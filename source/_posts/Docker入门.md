---
title: Docker入门
date: 2017-10-27 14:30:16
tags: Docker

---

在使用win的时候一直有个问题，很多软件需要用，但是使用频率很低，如果每个需要用的软件都装上的话，系统会变得非常庞大，后果就是导致系统运行起来非常缓慢，无奈只能重装系统了。在使用mac的时候，也遇到这种情况，而且删除软件的时候，不知道哪些文件改删除，哪些不改删除，慢慢的吞噬着硬盘的空间。近两年，微服务的概念兴起，偶然机会接触到docker，容器的概念，相当于虚拟机，但是对于传统虚拟机而言，启动和使用各方面都不尽相同，而且正好能够满足我的需求，于是在mac上安装docker来完成日常的开发工作，安装步骤如下。

1. 在mac上安装docker，下载地址https://www.docker.com/products/docker#/mac 下载安装即可
2. docker上有镜像，和容器的概念，镜像相当于系统盘，容器相当于运行的系统。如果要运行系统，首先需要有系统盘才行。
3. 下载images，我想运行ubuntu系统，可以使用 docker search ubuntu 来搜索，然后找到需要的系统后，docker pull ubuntu即可。但是很遗憾，你会发现下载失败，因为index.docker.io原因无法访问，所以只能使用国内的镜像文件，阿里云上有ubuntu的镜像文件，从aliyun上下载的ubuntu镜像。下载命令如下。需要制定域名地址 docker pull registry.cn-hangzhou.aliyuncs.com/acs/ubuntu
4. 下载以后可以通过 docker images 查看镜像文件    
5. 通过docker run -ti ubuntu /bin/bash 启动容器，你会发现已经进去了ubuntu系统。
6. 进入系统以后，需要先执行apt-get update来更新源文件，接下来就是安装各种需要的软件了
7. 退出出系统以后，可以通过命令 docker ps -a查看正在运行的容器。
8. 在容器中安装了很多软件，如果还是用原来的镜像启动的话，那么每次都需要重新安装了，所以可以生成自己的images，从正在运行的容器中去生成，命令如下。 docker commit 2d4c49bd7269 ubuntu:develop
9. 如果推出容器之后，想再进去，可以运行  docker start 2d4c49bd7269 来启动，然后 docker attach 2d4c49bd7269来再次进去容器中去.
10. 如果想查看正在运行中的容器，可以使用docker ps -a来查看。
11. 如果要关掉某个容器的话，需要使用docker stop  docker rm 2d4c49bd7269 即可
12. 如果需要访问docker内的地址，可以使用下面的命令 -p 8080:8080  建立端口映射，这样访问本机8080端口，便会打到docker内部了

上面这些命令，已经可以把docker玩转起来了，我在ubuntu中安装了mysql nginx redis等，简直就是理想的实验场地啊。
