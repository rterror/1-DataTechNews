# Run Docker Container On YARN
YARN 是大数据系统中的资源分配、任务调度系统，是最重要的组件之一。随着服务上云的推进，容器化、虚拟化也是当下趋势之一。
Docker是轻量级容器化技术的代表，container是其调度的基本单位，每个container拥有预先分配好的资源。将YARN和Docker结合也是顺利成章的事情。

YARN 引入Docker有两个过程：先是在ContainerExecutor抽象下实现了DockerContainerExecutor。ContainerExecutor的职能包括：

* Localizing (downloading and setting up) the resources required for running the container on any given node.
* Setting up the environment for the container to run (such as creating the directories for the container).
* Managing the life cycle of the YARN container (launching, monitoring, and cleaning up the container).

主要有三类：DefaultContainerExecutor, LinuxContainerExecutor, and WindowsSecureContainerExecutor。

但是这个设计有**实现**和**架构**上的硬伤：

* 实现上：用户无法指定image
* 架构上：每个NodeManager只能使用一个DockerContainerExecutor，这是通过node的配置决定的。一个DockerContainerExecutor限制了只能使用一种版本的软件。这导致无法支持普通的MR/Tez/Spark任务。此外，还需要在DockerContainerExecutor上重新实现一遍LinuxContainerExecutor。

因此，DockerContainerExecutor已经被弃用了。当前，在LinuxContainerExecutor之中支持container runtimes。container runtimes将ContainerExecutor分为两部分：**默认配置、运行时配置**。运行时配置可以随启动的container类型而变化。这种实现能够支持常规的MR/Tez/Spark任务。

运行时配置包括两种

* DefaultLinuxContainerRuntime：与YARN原来的行为一致
* DockerLinuxContainerRuntime：启动的是Docker 容器。 此实现在YARN-3611方案中。



[参考资料]

* https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.0/data-operating-system/content/run_docker_containers_on_yarn.html
