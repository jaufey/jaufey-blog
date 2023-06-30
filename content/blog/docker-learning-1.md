---
title: "Docker 小览（一）"
date: 2022-05-04T00:44:41+08:00
draft: false
tags: ["编程"]
categories: ["Docker"]
featured : true
summary: " "

---

本系列文章记录我学习 Docker 过程中的总结，疑问。


----

### 学习原因
- 无论是开发还是部署，Docker 使用起来方便快捷。技多不压身，希望亲身实践现代编程理念。
- 将来的一段时间内如果还在配置开发流程、安装依赖、调试环境上花费很多时间，是落伍的表现。
- 对于只限单机开发、单机部署的我来说，适应容器化、云原生时代的第一步就是学习 Docker，Docker 是后面**了解**运维、K8S 的基石。
- 大多框架、库的文档教程中提供了该项目容器化的实践方法。Docker 也可以是加速学习其他东西的一个垫脚石。
   - [针对开发的 Node Web APP 的容器化](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)
   - [Fastapi 的容器化部署](https://fastapi.tiangolo.com/deployment/docker/)

-----
### 以问题为导向的阐述
除非是某项特别难理解的技术，才需要画框图去区分每一个概念。  
得益于 Docker 体系的简洁性，此处的总结只限于罗列思路与思路的连接点。

1. 为了解决 **“分发什么”** 的问题，有了 image 的概念。  
   
   Docker image 是 build 出来的。开发完毕后，将代码库 build 为一个 image，即可 push 到 hub 上 分发。build 命令读取 Dockerfile 纯文本文件以构建镜像，Dockerfile 中指定当前程序的依赖，执行主任务前执行的操作，暴露的端口等。   
   
   让我们解析一个最基础的 build 命令：`docker build -t first-image .`  
      其中 `-t` 的意思是打 tag，表示生成镜像的名字；最后的这个 `.` 则表示 build 命令在当前目录下寻找 Dockerfile 文件。

1. 为了解决 **“运行什么”** 的问题，有了 container 的概念。
   
   Docker container 相当于进程，image 相当于代码。只有跑起来的 image 才是 container, 所以分发的对象是 image。run 命令则针对一个 image, 生成一个 container。  
   
   让我们解析一个最基础的 run 命令：`docker run -dp 3000:3000 getting-started`  
      其中 `-dp` 为 `-d` 和 `-p` 的连写， `-d` 为后台运行（detached），`-p` 为端口映射。
    
    **注意：在 docker 的配置或命令中，冒号左侧的都是相对于宿主机（host machine）而言的，冒号右侧的都是相对于容器内部而言的。**

2. 为了解决 **“容器内数据持久化”** 的问题，有了 volume 的概念。 
   
   volume 可将容器内的目录映射至宿主机，这样下次 container 启动时，加载出于宿主机上的 volume 即可，这样就不会产生数据丢失的问题了。

3. 为了解决 **“开发时同步代码”** 的问题，有了 bind mount 的概念。
   
   将宿主机的代码库映射到容器内，宿主机的代码改动会同步到运行中的容器内部，这样就可以达到 “代码库在宿主机，依赖在容器内” 的目的。

4. 为了实践 **“复用容器，单一职责”** 的理念，一个项目可能用到多个容器（比如一个项目可包含 app 容器，MySQL 容器，Redis 容器等）
   
   所以一个问题产生了——**“如何解决多容器间的通信问题？”**。由此，docker 有了 Network 的概念。
   处于同一 Network 中的容器可相互通信，比如 app 容器要连接 MySQL 容器，则使他们共处同一个网络即可。

5. 为了缓解 **“多容器项目带来的复杂性”**，比如开启项目前，我们需要执行创建 volume、创建网络、设置环境变量等需要严密配合的任务，容易出错而且不易追踪到底执行了什么。由此，docker 有了 docker-compose 指令。
   这个指令读取 docker-compose.yml，接管整个项目的运行。docker-compose.yml 内可声明前面提到的所有内容，最终运行起来的 contaienr 呈现为 group 的形式。
   
   
-----

### 问题
1. 注意 docker-compose 跟容器编排没关系，不要顾名思义。编排的英文是 “Orchestration” 这个单词。  
   
   docker-compose 只解决多容器及配套服务的启动问题，在生产环境中，没有人会直接通过 docker run 或 docker-compose up 来运行一个项目，因为除了容器的正常运行之外，我们还需要考虑很多问题。

   容器编排指的是**自动化**容器的部署、管理、扩展和联网，所以，容器编排工具（Kubernetes, Swarm, Nomad, and ECS）是容器的管理者，是容器的上层建筑。

2. 

-----

### 参考资料
- [Docker 官网文档十步曲](https://docs.docker.com/get-started/)

