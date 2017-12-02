# Docker方式部署Jenkins

1.下载Jenkins的docker镜像：

docker pull hub.c.163.com/library/jenkins:latest

2.启动Jenkins镜像：

docker run -itd -u root --name jenkins -p 8080:8080 -p 50000:50000 –v /var/jenkins\_home:/var/jenkins\_home  hub.c.163.com/library/jenkins

3.进入Jenkins容器

docker exec -it jenkins bash

4.访问Jenkins服务：

[http://IP:8080/jenkins](http://IP:8080/jenkins)

# Jenkins性能调优实践

* 使用Pipeline方式比配置方式运行更快。
* 一个代码库，一条流水线。
* 让Pipeline做中间组织的工作，而不是取代其他工具。
* 能用脚本实现的，就不要用插件。
* 根据资源情况限制并发执行的job数量。
* 使用共享库抽象公共的代码并持续优化。
* 不要写太复杂的Pipeline脚本（&gt;1000行），包括共享库代码在内。
* 不要对执行环境有强依赖（如外网环境、数据库等）。
* 使用“@NonCPS”注解高耗代码方法。
* 使用SSD硬盘等高性能硬件。
* COMING SOON:Durability/Speed Options

![](/assets/import1401.png)![](/assets/import1402.png)

# Jenkins构建集群实践

**Jenkins Master-Slave集群**

* Slave编译环境不统一
* Slave资源利用不均衡
* Slave的workspace空间浪费

![](/assets/import1403.png)

**基于K8s的统一调度集群**

* 使用Docker image标准化Jenkins环境
* 创建pod 挂载容器 slave
* 容器Slave按需弹性收缩

![](/assets/import1404.png)

