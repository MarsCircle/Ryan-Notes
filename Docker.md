# Docker

-----------------------------------
``` shell
~$ sudo docker version　#查看本地docker的版本
~$ sudo docker image ls　#查看本地的镜像都有啥子
```

## Image

* 分层的，每一层都可以添加改变删除文件，成为一个新的镜像
* **Image本身是只读的，不可修改**

## 怎么获得镜像

* Build from Dockerfile 

* Pull from Registry

``` shell
~$ sudo docker pull ubuntu:14.04
```

 这里从Docker hub 拉镜像的时候由于翻不了墙的原因，可能会失败，可以改为国内的源自己查一下

### 给非管理员用户赋予使用Docker的权限

``` shell
~$ sudo groupadd docker
~$ sudo gpasswd -a ryan docker 　　# 这里的 ryan 是我的用户名，这里应该替换成自己的用户名
~$ sudo service docker restart 　　　#重启 Docker 服务
~$ docker image ls                                      #这样所有命令不加 sudo 也可以了
```

### 这里我们尝试使用 Dockerfile 来尝试自制一个镜像

````shell
~$ docker pull hello-world 　#先从官方拉一个简单的 hello-world 镜像
~$ docker run hello-world      #运行一下这个镜像
````

```shell
ryan@ryan:~$ docker run hello-world #结果展示

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

DIY一个镜像开始！

```shell
~$ mkdir hello-world  #先在本地建一个目录
~$ cd hello-world
~/hello-world$ vim hello.c　# 写一个C语言的hello-world(内容在下方)
~/hello-world$ gcc -static hello.c -o hello  # 把他变成一个可执行文件hello
~/hello-world$ ./hello #运行一下，结果输出hello-world
~/hello-world$ vim Dockerfile #建立一个Dockerfile（内容在下方）
~/hello-world$ docker build -t my-hello-world . #注意最后有一个 .     这里建立一个my-hello-world的镜像(.表示在当前目录build)
~/hello-world$ docker run my-hello-world #这里运行一下 ，结果输出hello-world
```

hello.c 如下

``` c
#include<stdio.h>

int main()
{       
        printf("hello world\n");
}    
```

Dockerfile 内容如下（因为要创建一个自己的镜像所以FROM scratch，相当于不基于任何镜像）

```dockerfile
FROM scratch
ADD hello /
CMD ["/hello"]
```

看Dockerfile的命令

```shell
$ more hello-world/Dockerfile #more 加 路径
```



-----------------------------

## Container

* 通过镜像生成的，在镜像(Image layer)的基础上建立一层可以读写的容器层(container layer)

### 创建容器：

``` shell
$ docker run my-hello-world 
```

这里运行了之前创建的一个镜像, 其实是系统根据该镜像自行建立了一个容器，并运行之后自动退出

```shell
$ docker run ubuntu:14.04
```

试一下运行Ubuntu的官方镜像，这里会发现没有任何结果，我们通过查询命令看一下

``` shell
$ docker container ls  #查询正在运行的容器
$ docker container ls -a #查询所有容器的状态
```

```` shell
ryan@ryan:~$ docker container ls -a
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                      PORTS               NAMES
836e172855e2        ubuntu                  "/bin/bash"         5 minutes ago       Exited (0) 5 minutes ago            fervent_clarke

````

就可以查到之前run的一个镜像，其实是Docker先创建的容器，运行之后直接退出

怎么进入这个容器进行操作呢？

````shell
$ docker run -it ubuntu:14.04 #这样就可以进入这个容器去配置了
````

怎么删除容器呢？

```shell
$ docker container rm 容器的id # 不写全也可以貌似
```

这些命令都有简化版

```shell
$ docker container rm 容器的id 
$ docker  rm 容器的id # 默认对容器操作

$ docker image rm 镜像名/id #删除镜像的操作
$ docker rmi 镜像名/id 

$ docker image ls
$ docker images

$ docker container ls -a
$ docker ps -a
```

如果玩着玩着创建了一堆容器，怎么一起删

````shell
$ docker container ls -aq  #查询所有容器ID
$ docker container ls -a | awk {'print$1'} #打印第一列，和上一个效果差不多

#咋能一起删呢
$ docker rm $(docker container ls -aq)

#要是有的想要咋办(比如正在运行的容器不删)
$ docker container ls -f "status=exited" #能筛选出所有已经退出的容器
$ docker container ls -f "status=exited" -q #能筛选出所有已经退出的容器的ID
$ docker rm $(docker container ls -f "status=exited" -q) #所以未在工作的容器就被删除了

````

### 那么如果自己创建一个容器之后，觉得配置的很好，可不可以把容器变成镜像呢？当然可

```shell
$ docker image ls #先看一下都有什么镜像
$ docker run -it ubuntu #以ubuntu为例建立容器
$ sudo apt-get update #更新一下容器本身
$ sudo apt-get install vim #给容器安装一个vim
$ exit #退出容器

$ docker commit 容器的名字  要生成镜像的名字
$ docker  history 镜像id  #可以看到镜像是由哪几层形成的
```

或者建立一个Dockerfile来建立镜像

```shell
$ mkdir docker-ubuntu-vim
$ cd docker-ubuntu-vim
$ vim Dockerfile
$ docker build -t ubuntu-vim-new .
```

Dockerfile 内容如下

```dockerfile
FROM ubuntu
RUN sudo apt-get install -y vim
```

Dockerfile 都有些什么语法呢

```dockerfile
FROM: 
FROM scratch #制作base image
FROM ubuntu #使用base image

LABEL: 类比于代码里的注释 image的Metadata
LABEL maintainer = "amazingryan@163.com"
LABEL version = "1.0"
LABEL description = "This is description"

RUN : 每运行一次RUN,就会多一个分层，所以尽量使用反斜线\ 在一行写完
RUN apt-get update && apt-get install -y perl \
		   pwgen --no-install-recommends && rm-rf \
		   /var/lib/apt/lists/*
		   
WORKDIR :设定当前工作目录的,尽量用WORKDIR,不要用RUN cd,尽量使用绝对目录
WORKDIR /test  #如果没有会自动创建test目录
WORKDIR demo
RUN pwd　　　#输出结果应该是/test/demo

ADD and COPY :大部分情况COPY 优于ADD,ADD除了COPY还能解压
ADD hello /  　#把本地的hello文件添加到根目录/里
ADD test.tar.gz / #ADD优点是不仅会添加到根目录还会顺便解压

WORKDIR /root
ADD hello test/   # /root/test/hello

WORKDIR /root
COPY hello test/   # /root/test/hello

如果添加远程的目录/文件　要使用RUN curl或者wget

ENV:设置常量
ENV MYSQL_VERSION 5.6 #设置常量
RUN apt-get install -y mysql-server = "${MYSQL_VERSION}"  \
&& rm -rf /var/lib/apt/lists/* #引用常量

VOLUME
EXPOSE
RUN:　执行命令并创建新的Image Layer

ENTRYPOINT :设置容器启动时运行的命令
让容器以应用程序或者服务的形式运行
不会被忽略，一定会执行

CMD：设置容器启动后默认执行的命令和参数
容器启动时默认执行的命
如果docker run 指定了其它命令，CMD命令会被忽略　例如Docker run -it [image] /bin/bash
如果定义了多个CMD ,只有最后一个会执行


Shell格式
RUN apt-get install -y vim
CMD echo "hello docker"
ENTRYPQINT echo "hello docker"

Exec格式
RUN ["apt-get" , "install" , "-y" , "vim"]
CMD[" /bin/echo" , "hello docker"]
ENTRYPOINT ["/bin/echo" , "hello docker"]

```

[Docker hub里绝大多数的官方镜像都有对应的Dokcerfile](https://github.com/docker-library/docs)，可以参考一下

这里有两个例子　Dockerfile

  ```dockerfile
FROM ubuntu
ENV name Docker
ENTRYPOINT echo "hello $name"
  ```

```dockerfile
FROM ubuntu
ENV name Docker
ENTRYPOINT [ "/bin/echo" , "hello $name" ]
```

```shell
$ vim Dockerfile
$ docker build -t ubuntu-shell .
$ vim Dockerfile #修改为exec模式
$ docker build -t ubuntu-exec .
$ docker run ubuntu-shell  #结果为hello docker
$ docker run ubuntu-exec  #结果为hello $name,因为exec模式本质上不是在shell执行的　，而是单纯的执行echo命令
```

```dockerfile
FROM ubuntu
ENV name Docker
ENTRYPOINT [ "/bin/bash" ,"-c" ,"echo hello $name" ] #-c表示后面的是命令的参数
```

```dockerfile

```




