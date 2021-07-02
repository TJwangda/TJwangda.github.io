[TOC]

# 命令

## 基础

~~~shell
卸载旧版本：
yum remove docker docker-common docker-selinux docker-engine 
yum remove docker-ce
卸载后将保留 /var/lib/docker 的内容（镜像、容器、存储卷和网络等）。
rm -rf /var/lib/docker
1.安装依赖软件包
yum install -y yum-utils device-mapper-persistent-data lvm2 
#安装前可查看device-mapper-persistent-data和lvm2是否已经安装 
rpm -qa|grep device-mapper-persistent-data 
rpm -qa|grep lvm2
2.设置yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
3.更新yum软件包索引
yum makecache fast
二、安装最新版本docker-ce
yum install docker-ce -y 
#安装指定版本docker-ce可使用以下命令查看 
yum list docker-ce.x86_64 --showduplicates | sort -r 
# 安装完成之后可以使用命令查看 
docker version
三、配置镜像加速
这里使用阿里云的免费镜像加速服务，也可以使用其他如时速云、网易云等
1.注册登录开通阿里云容器镜像服务
2.查看控制台，招到镜像加速器并复制自己的加速器地址
3.找到/etc/docker目录下的daemon.json文件，没有则直接 vi daemon.json
4.加入以下配置
#填写自己的加速器地址 
{ 
"registry-mirrors": ["https://zfzbet67.mirror.aliyuncs.com"] 
}
5.通知systemd重载此配置文件；
systemctl daemon-reload
6.重启docker服务
systemctl restart docker
~~~

## 常用命令

~~~shell
输入 docker 可以查看Docker的命令用法，输入 docker COMMAND --help 查看指定命令详细用法。

镜像常用操作
查找镜像：
docker search 关键词 #搜索docker hub网站镜像的详细信息
下载镜像：
docker pull 镜像名:TAG # Tag表示版本，有些镜像的版本显示latest，为最新版本
查看镜像：
docker images # 查看本地所有镜像
删除镜像：
docker rmi -f 镜像ID或者镜像名:TAG # 删除指定本地镜像 # -f 表示强制删除
获取元信息：
docker inspect 镜像ID或者镜像名:TAG # 获取镜像的元信息，详细信息

容器常用操作
运行：
docker run --name 容器名 -i -t -p 主机端口:容器端口 -d -v 主机目录:容器目录:ro 镜像ID或镜像名:TAG # --name 指定容器名，可自定义，不指定自动命名 
# -i 以交互模式运行容器 
# -t 分配一个伪终端，即命令行，通常-it组合来使用 
# -p 指定映射端口，讲主机端口映射到容器内的端口 
# -d 后台运行容器 
# -v 指定挂载主机目录到容器目录，默认为rw读写模式，ro表示只读
容器列表：
docker ps -a -q 
# docker ps查看正在运行的容器 
# -a 查看所有容器（运行中、未运行） 
# -q 只查看容器的ID
启动容器：
docker start 容器ID或容器名
停止容器：
docker stop 容器ID或容器名
删除容器：
docker rm -f 容器ID或容器名 
# -f 表示强制删除
查看日志：
docker logs 容器ID或容器名
进入正在运行容器：
docker exec -it 容器ID或者容器名 /bin/bash 
# 进入正在运行的容器并且开启交互模式终端 
# /bin/bash是固有写法，作用是因为docker后台必须运行一个进程，否则容器就会退出，在这里表示启动容器后启动 bash。 
# 也可以用docker exec在运行中的容器执行命令
退出容器不停止容器
ctrl+P+Q
拷贝文件：
docker cp 主机文件路径 容器ID或容器名:容器路径 
#主机中文件拷贝到容器中 docker cp 容器ID或容器名:容器路径 主机文件路径 
#容器中文件拷贝到主机中
获取容器元信息：
docker inspect 容器ID或容器名
~~~

![image-20200720102231102](E:\dev\picture\image-20200720102231102.png)

## Docker生成镜像的两种方式

有时候从Docker镜像仓库中下载的镜像不能满足要求，我们可以基于一个基础镜像构建一个自己的镜像

两种方式：

+ 更新镜像：使用 docker commit 命令

+ 构建镜像：使用 docker build 命令，需要创建Dockerfifile文件

### **更新镜像**

先使用基础镜像创建一个容器，然后对容器内容进行更改，然后使用 docker commit 命令提交为一个新的镜像（以tomcat为例）。

1.根据基础镜像，创建容器

~~~shell
docker run --name mytomcat -p 80:8080 -d tomcat
~~~

2.修改容器内容

~~~shell
docker exec -it mytomcat /bin/bash 
cd webapps/ROOT 
rm -f index.jsp 
echo hello world > index.html 
exit
~~~

3.提交为新镜像

~~~shell
docker commit -m="描述消息" -a="作者" 容器ID或容器名 镜像名:TAG 
# 例: 
# docker commit -m="修改了首页" -a="华安" mytomcat huaan/tomcat:v1.0
~~~

4.使用新镜像运行容器

~~~shell
docker run --name tom -p 8080:8080 -d huaan/tomcat:v1.0
~~~

### **使用Dockerfifile构建镜像**、

**使用Dockerfifile构建SpringBoot应用镜像**

一、准备

1.把你的springboot项目打包成可执行jar包

2.把jar包上传到Linux服务器

二、构建

1.在jar包路径下创建Dockerfifile文件 vi Dockerfile

~~~shell
# 指定基础镜像，本地没有会从dockerHub pull下来 
FROM java:8 
#作者 
MAINTAINER huaan 
# 把可执行jar包复制到基础镜像的根目录下 
ADD luban.jar /luban.jar 
# 镜像要暴露的端口，如要使用端口，在执行docker run命令时使用-p生效 
EXPOSE 80 
# 在镜像运行为容器后执行的命令 
ENTRYPOINT ["java","-jar","/luban.jar"]
~~~

2.使用 docker build 命令构建镜像，基本语法

~~~shell
docker build -t huaan/mypro:v1 . 
# -f指定Dockerfile文件的路径 
# -t指定镜像名字和TAG 
# .指当前目录，这里实际上需要一个上下文路径
~~~

三、运行

运行自己的SpringBoot镜像

~~~shel
docker run --name pro -p 80:80 -d 镜像名:TAG
~~~

### **Dockerfifile常用指令**

#### **FROM**

FROM指令是最重要的一个并且必须为Dockerfifile文件开篇的第一个非注释行，用于为镜像文件构建过程指定基础镜像，后续的指令运行于此基础镜像提供的运行环境

这个基础镜像可以是任何可用镜像，默认情况下docker build会从本地仓库找指定的镜像文件，如果不存在就会从Docker Hub上拉取

语法

~~~shell
FROM <image> 
FROM <image>:<tag> 
FROM <image>@<digest>
~~~

#### **MAINTAINER(depreacted)**

Dockerfifile的制作者提供的本人详细信息

Dockerfifile不限制MAINTAINER出现的位置，但是推荐放到FROM指令之后

语法：

~~~shel
MAINTAINER <name>
~~~

name可以是任何文本信息，一般用作者名称或者邮箱

#### **LABEL**

给镜像指定各种元数据

语法：

~~~shell
LABEL <key>=<value> <key>=<value> <key>=<value>...
~~~

一个Dockerfifile可以写多个LABEL，但是不推荐这么做，Dockerfifile每一条指令都会生成一层镜像，如果LABEL太长可以使用\符号换行。构建的镜像会继承基础镜像的LABEL，并且会去掉重复的，但如果值不同，则后面的值会覆盖前面的值。

#### **COPY**

用于从宿主机复制文件到创建的新镜像文件

语法：

~~~shell
COPY <src>...<dest> COPY ["<src>",..."<dest>"] 
# <src>：要复制的源文件或者目录，可以使用通配符 
# <dest>：目标路径，即正在创建的image的文件系统路径；建议<dest>使用绝对路径，否则COPY指令则以WORKDIR为 其起始路径
~~~

注意：如果你的路径中有空白字符，通常会使用第二种格式

规则：

+ <src> 必须是build上下文中的路径，不能是其父目录中的文件

+ 如果 <src> 是目录，则其内部文件或子目录会被递归复制，但 <src> 目录自身不会被复制

+ 如果指定了多个 <src> ，或在 <src> 中使用了通配符，则 <dest> 必须是一个目录，则必须以/符号结尾

+ 如果 <dest> 不存在，将会被自动创建，包括其父目录路径

#### **ADD**

基本用法和COPY指令一样，ADD支持使用TAR文件和URL路径

语法：

~~~shell
ADD <src>...<dest> 
ADD ["<src>",..."<dest>"]
~~~

规则：

+ 和COPY规则相同

+ 如果 <src> 为URL并且 <dest> 没有以/结尾，则 <src> 指定的文件将被下载到 <dest>

+ 如果 <src> 是一个本地系统上压缩格式的tar文件，它会展开成一个目录；但是通过URL获取的tar文件不会自动展开

+ 如果 <src> 有多个，直接或间接使用了通配符指定多个资源，则 <dest> 必须是目录并且以/结尾

#### **WORKDIR**

用于为Dockerfifile中所有的RUN、CMD、ENTRYPOINT、COPY和ADD指定设定工作目录，只会影响当前WORKDIR之后的指令。

语法：

~~~shell
WORKDIR <dirpath>
~~~

在Dockerfifile文件中，WORKDIR可以出现多次，路径可以是相对路径，但是它是相对于前一个WORKDIR指令指定的路径

另外，WORKDIR可以是ENV指定定义的变量

#### **VOLUME**

用来创建挂载点，可以挂载宿主机上的卷或者其他容器上的卷

语法：

~~~shell
VOLUME <mountpoint> 
VOLUME ["<mountpoint>"]
~~~

不能指定宿主机当中的目录，宿主机挂载的目录是自动生成的

#### **EXPOSE**

用于给容器打开指定要监听的端口以实现和外部通信

语法：

~~~shell
EXPOSE <port>[/<protocol>] [<port>[/<protocol>]...]
~~~

<protocol> 用于指定传输层协议，可以是TCP或者UDP，默认是TCP协议

EXPOSE可以一次性指定多个端口，例如： EXPOSE 80/tcp 80/udp

#### **ENV**

用来给镜像定义所需要的环境变量，并且可以被Dockerfifile文件中位于其后的其他指令(如ENV、ADD、COPY等)所调用，调用格式：$variable_name或者${variable_name}

语法：

~~~shell
ENV <key> <value> 
ENV <key>=<value>...
~~~

第一种格式中， <key> 之后的所有内容都会被视为 <value> 的组成部分，所以一次只能设置一个变量

第二种格式可以一次设置多个变量，如果 <value> 当中有空格可以使用\进行转义或者对 <value> 加引号进行标识；

另外\也可以用来续行

#### **ARG**

用法同ENV

语法

~~~shell
ARG <name>[=<default value>]
~~~

指定一个变量，可以在docker build创建镜像的时候，使用 --build-arg <varname>=<value> 来指定参数

#### **RUN**

用来指定docker build过程中运行指定的命令

语法：

~~~shell
RUN <command> 
RUN ["<executable>","<param1>","<param2>"]
~~~

第一种格式里面的参数一般是一个shell命令，以 /bin/sh -c 来运行它

第二种格式中的参数是一个JSON格式的数组，当中 <executable> 是要运行的命令，后面是传递给命令的选项或者参数；但是这种格式不会用 /bin/sh -c 来发起，所以常见的shell操作像变量替换和通配符替换不会进行；如果你运行的命令依赖shell特性，可以替换成类型以下的格式

~~~shell
RUN ["/bin/bash","-c","<executable>","<param1>"]
~~~

#### **CMD**

容器启动时运行的命令

语法：

~~~shell
CMD <command> 
CMD ["<executable>","<param1>","<param2>"] 
CMD ["<param1>","<param2>"]
~~~

前两种语法和RUN相同

第三种语法用于为ENTRYPOINT指令提供默认参数

RUN和CMD区别：

+ RUN指令运行于镜像文件构建过程中，CMD则运行于基于Dockerfifile构建出的新镜像文件启动为一个容器的时

候

+ CMD指令的主要目的在于给启动的容器指定默认要运行的程序，且在运行结束后，容器也将终止；不过，CMD命令可以被docker run的命令行选项给覆盖

+ Dockerfifile中可以存在多个CMD指令，但是只有最后一个会生效

#### **ENTRYPOINT**

类似于CMD指令功能，用于给容器指定默认运行程序

语法：

~~~shell
ENTRYPOINT<command> 
ENTRYPOINT["<executable>","<param1>","<param2>"]
~~~

和CMD不同的是ENTRYPOINT启动的程序不会被docker run命令指定的参数所覆盖，而且，这些命令行参数会被当做参数传递给ENTRYPOINT指定的程序(但是，docker run命令的--entrypoint参数可以覆盖ENTRYPOINT)

docker run命令传入的参数会覆盖CMD指令的内容并且附加到ENTRYPOINT命令最后作为其参数使用

同样，Dockerfifile中可以存在多个ENTRYPOINT指令，但是只有最后一个会生效

Dockerfifile中如果既有CMD又有ENTRYPOINT，并且CMD是一个完整可执行命令，那么谁在最后谁生效

#### **ONBUILD**

用来在Dockerfifile中定义一个触发器

语法：

~~~shell
ONBUILD <instruction>
~~~

Dockerfifile用来构建镜像文件，镜像文件也可以当成是基础镜像被另外一个Dockerfifile用作FROM指令的参数

在后面这个Dockerfifile中的FROM指令在构建过程中被执行的时候，会触发基础镜像里面的ONBUILD指令

ONBUILD不能自我嵌套，ONBUILD不会触发FROM和MAINTAINER指令

在ONBUILD指令中使用ADD和COPY要小心，因为新构建过程中的上下文在缺少指定的源文件的时候会失败

## **Docker** **网络**

Docker允许通过外部访问容器或容器互联的方式来提供网络服务。

安装Docker时，会自动安装一块Docker网卡称为docker0，用于Docker各容器及宿主机的网络通信，网段为

172.0.0.1。

Docker网络中有三个核心概念：沙盒（Sandbox）、网络（Network）、端点（Endpoint）。

+ 沙盒，提供了容器的虚拟网络栈，也即端口套接字、IP路由表、防火墙等内容。隔离容器网络与宿主机网络，形成了完全独立的容器网络环境。

+ 网络，可以理解为Docker内部的虚拟子网，网络内的参与者相互可见并能够进行通讯。Docker的虚拟网络和宿主机网络是存在隔离关系的，其目的主要是形成容器间的安全通讯环境。

+ 端点，位于容器或网络隔离墙之上的洞，主要目的是形成一个可以控制的突破封闭的网络环境的出入口。当容器的端点与网络的端点形成配对后，就如同在这两者之间搭建了桥梁，便能够进行数据传输了。

### **Docker的四种网络模式**

Docker服务在启动的时候会创建三种网络，bridge、host和none，还有一种共享容器的模式container

**Bridge**

桥接模式，主要用来对外通信的，docker容器默认的网络使用的就是bridge。

使用bridge模式配置容器自定的网络配置

~~~shell
# 配置容器的主机名 
docker run --name t1 --network bridge -h [自定义主机名] -it --rm busybox 
# 自定义DNS 
docker run --name t1 --network bridge --dns 114.114 -it --rm busybox 
# 给host文件添加一条 
docker run --name t1 --network bridge --add-host [hostname]:[ip] -it --rm busybox
~~~

**Host**

host类型的网络就是主机网络的意思，绑定到这种网络上面的容器，内部使用的端口直接绑定在主机上对应的端口，而如果容器服务没有使用端口，则无影响。

**None**

从某种意义上来说，none应该算不上网络了，因为它不使用任何网络，会形成一个封闭网络的容器

**container**

共享另外一个容器的network namespace，和host模式差不多，只是这里不是使用宿主机网络，而是使用的容器网络

**开放端口**

Docker0为NAT桥，所以容器一般获得的是私有网络地址

给docker run命令使用-p选项即可实现端口映射，无需手动添加规则

- -p 选项的使用

  ​	-p <containerPort>

  ​			将指定的容器端口映射到主机所有地址的一个动态端口

​          -p <hostPort>:<containerPort>

​					将容器端口 <containerPort> 映射到指定的主机端口 <hostPort> 

​			-p <ip>::<containerPort>

​					将指定的容器端口 <containerPort> 映射到主机指定 <ip> 的动态端口

​			-p <ip>:<hostPort>:<containerPort>

​					将指定的容器端口 <containerPort> 映射至主机指定 <ip> 的端口 <hostPort>

+ 动态端口指随机端口，可以使用docker port命令查看具体映射结果

+ -P 暴露所有端口（所有端口指构建镜像时EXPOSE的端口）

自定义docker0桥的网络属性信息：/etc/docker/daemon.json文件

~~~shell
{"bip": "192.168.1.5/24",
"fixed-cidr": "10.20.0.0/16", 
"fixed-cidr-v6": "2001:db8::/64",
"mtu": 1500, "default-gateway": "10.20.1.1",
"default-gateway-v6": "2001:db8:abcd::89", 
"dns": ["10.20.1.2","10.20.1.3"] 
}
~~~

核心选项为bip，即bridge ip之意，用于指定docker0桥自身的IP地址；其它选项可通过此地址计算得出

远程连接

创建自定义的桥

~~~shell
docker network create -d bridge --subnet "172.26.0.0/16" --gateway "172.26.0.1" mybr0
~~~

### **Docker Compose**

从上一节课我们了解到可以使用一个Dockerfifile模板文件来快速构建一个自己的镜像并运行为应用容器。但是在平时工作的时候，我们会碰到多个容器要互相配合来使用的情况，比如数据库加上咱们Web应用等等。这种情况下，每次都要一个一个启动容器设置命令变得麻烦起来，所以Docker Compose诞生了。

**简介**

Compose的作用是“定义和运行多个Docker容器的应用”。使用Compose，你可以在一个配置文件（yaml格式）中配置你应用的服务，然后使用一个命令，即可创建并启动配置中引用的所有服务。

Compose中两个重要概念：

+ 服务 (service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

+ 项目 (project)：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml文件中定义。

**安装**

Compose支持三平台Windows、Mac、Linux，安装方式各有不同。我这里使用的是Linux系统，其他系统安装方法可以参考官方文档和开源GitHub链接：

Docker Compose官方文档链接：https://docs.docker.com/compose

Docker Compose GitHub链接：https://github.com/docker/compose

Linux上有两种安装方法，Compose项目是用Python写的，可以使用Python-pip安装，也可以通过GitHub下载二进制文件进行安装。

**通过Python-pip安装**

1.安装Python-pip

~~~shell
yum install -y epel-release yum install -y python-pip
~~~

2.安装docker-compose

~~~shell
pip install docker-compose
~~~

3.验证是否安装

~~~shell
docker-compose version
~~~

4.卸载

~~~shell
pip uninstall docker-compose
~~~

**通过GitHub链接下载安装**

非ROOT用户记得加sudo

1.通过GitHub获取下载链接，以往版本地址：https://github.com/docker/compose/releases

~~~shell
curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
~~~

2.给二进制下载文件可执行的权限

~~~shell
chmod +x /usr/local/bin/docker-compose
~~~

3.可能没有启动程序，设置软连接，比如:

~~~shell
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
~~~

4.验证是否安装

~~~shell
docker-compose version
~~~

5.卸载

如果是二进制包方式安装的，删除二进制文件即可。

~~~shell
rm /usr/local/bin/docker-compose
~~~

**简单实例**

Compose的使用非常简单，只需要编写一个docker-compose.yml，然后使用docker-compose 命令操作即可。

docker-compose.yml描述了容器的配置，而docker-compose 命令描述了对容器的操作。

1.我们使用一个微服务项目先来做一个简单的例子，首先创建一个compose的工作目录，然后创建一个eureka文件夹，里面放可执行jar包和编写一个Dockerfifile文件，目录结构如下：

~~~shell
compose 
	eureka
		Dockerfile 
			eureka-server-2.0.2.RELEASE.jar
~~~

2.在compose目录创建模板文件docker-compose.yml文件并写入以下内容：

~~~shell
version: '1' 
services:
	eureka: 
        build: ./eureka 
        ports: 
         - 3000:3000 	
        expose: 
         - 3000
~~~

**Docker Compose模板文件常用指令**

**image**

指定镜像名称或者镜像id，如果该镜像在本地不存在，Compose会尝试pull下来。

示例：

image: java:8

**build**

指定Dockerfifile文件的路径。可以是一个路径，例如：

build: ./dir

也可以是一个对象，用以指定Dockerfifile和参数，例如：

build: context: ./dir dockerfifile: Dockerfifile-alternate args: buildno: 1

**command**

覆盖容器启动后默认执行的命令。

示例：

command: bundle exec thin -p 3000

也可以是一个list，类似于Dockerfifile总的CMD指令，格式如下：

command: [bundle, exec, thin, -p, 3000]

**links**

链接到其他服务中的容器。可以指定服务名称和链接的别名使用SERVICE:ALIAS 的形式，或者只指定服务名称，示例：

web: links: - db - db:database - redis

**external_links**

表示链接到docker-compose.yml外部的容器，甚至并非Compose管理的容器，特别是对于那些提供共享容器或共同服务。格式跟links类似，示例：

external_links: - redis_1 - project_db_1:mysql - project_db_1:postgresql

**ports**

暴露端口信息。使用宿主端口:容器端口的格式，或者仅仅指定容器的端口（此时宿主机将会随机指定端口），类似

于docker run -p ，示例：

ports:

+ "3000"

+ "3000-3005"

+ "8000:8000"

+ "9090-9091:8080-8081"

+ "49100:22"

+ "127.0.0.1:8001:8001"

+ "127.0.0.1:5000-5010:5000-5010"

**expose**

暴露端口，只将端口暴露给连接的服务，而不暴露给宿主机，示例：

expose: - "3000" - "8000"

**volumes**

卷挂载路径设置。可以设置宿主机路径 （HOST:CONTAINER） 或加上访问模式 （HOST:CONTAINER:ro）。示例：

volumes:

Just specify a path and let the Engine create a volume

+ /var/lib/mysql

Specify an absolute path mapping

+ /opt/data:/var/lib/mysql

Path on the host, relative to the Compose fifile

+ ./cache:/tmp/cache

User-relative path

+ ~/confifigs:/etc/confifigs/:ro

Named volume

+ datavolume:/var/lib/mysq

**volumes_from**

从另一个服务或者容器挂载卷。可以指定只读或者可读写，如果访问模式没有指定，则默认是可读写。示例：

volumes_from:

+ service_name

+ service_name:ro

+ container:container_name

+ container:container_name:rw

**environment**

设置环境变量。可以使用数组或者字典两种方式。只有一个key的环境变量可以在运行Compose的机器上找到对应的值，这有助于加密的或者特殊主机的值。示例：

environment: RACK_ENV: development SHOW: 'true' SESSION_SECRET: environment: -

RACK_ENV=development - SHOW=true - SESSION_SECRET

**env_fifile**

从文件中获取环境变量，可以为单独的文件路径或列表。如果通过 docker-compose -f FILE 指定了模板文件，则env_fifile 中路径会基于模板文件路径。如果有变量名称与 environment 指令冲突，则以envirment 为准。示例：

env_fifile: .env env_fifile: - ./common.env - ./apps/web.env - /opt/secrets.env

**extends**

继承另一个服务，基于已有的服务进行扩展。

**net**

设置网络模式。示例：

net: "bridge" net: "host" net: "none" net: "container:[service name or container name/id]"

**dns**

配置dns服务器。可以是一个值，也可以是一个列表。示例：

dns: 8.8.8.8 dns: - 8.8.8.8 - 9.9.9.9

**dns_search**

配置DNS的搜索域，可以是一个值，也可以是一个列表，示例：

dns_search: example.com dns_search: - dc1.example.com - dc2.example.com

**其它**

docker-compose.yml 还有很多其他命令，可以参考docker-compose.yml文件官方文档：

https://docs.docker.com/compose/compose-fifile/

**使用Docker Compose编排SpringCloud微服务**

使用docker-compose一次性来编排三个微服务:eureka服务(eureka-server-2.0.2.RELEASE.jar)、user服务(user-2.0.2.RELEASE.jar)、power服务(power-2.0.2.RELEASE.jar)

1.创建一个工作目录和docker-compose模板文件

2.工作目录下创建三个文件夹eureka、user、power，并分别构建好三个服务的镜像文件

以eureka的Dockerfifile为例

~~~shell
# 基础镜像 
FROM java:8 
# 作者 
MAINTAINER huaan 
# 把可执行jar包复制到基础镜像的根目录下 
ADD eureka-server-2.0.2.RELEASE.jar /eureka-server-2.0.2.RELEASE.jar 
# 镜像要暴露的端口，如要使用端口，在执行docker run命令时使用-p生效 
EXPOSE 3000 
# 在镜像运行为容器后执行的命令 
ENTRYPOINT ["java","-jar","/eureka-server-2.0.2.RELEASE.jar"]
~~~

目录文件结构：

~~~shell
compose 
        docker-compose.yml 
        eureka 
            Dockerfile 
            eureka-server-2.0.2.RELEASE.jar 
        user 
            Dockerfile 
            user-2.0.2.RELEASE.jar 
		power 
            Dockerfile 
            power-2.0.2.RELEASE.jar
~~~

3.编写docker-compose模板文件：

~~~shell
version: '1' 
services: 
	eureka: 
        image: eureka:v1 
        ports: 
        	- 8080:8080 
    user: 
    	image: user:v1 
    	ports: 
    		- 8081:8081 
    power: 
    	image: power:v1 
    	ports: 
    		- 8082:8082
~~~

4.启动微服务，可以加上参数-d后台启动

~~~shell
docker-compose up -d
~~~

## **将本地镜像发布到阿里云**

有时候需要共享镜像或者习惯使用自己定义的镜像，可以注册私有仓库，国内推荐使用阿里云

步骤：

1.登录阿里云容器镜像服务：https://cr.console.aliyun.com/cn-hangzhou/repositories

2.将镜像推送到阿里云

~~~shell
# 登录阿里云的docker仓库 
$ sudo docker login --username=[用户名] registry.cn-hangzhou.aliyuncs.com 
# 创建指定镜像的tag，归入某个仓库 
$ sudo docker tag [镜像ID] registry.cn-hangzhou.aliyuncs.com/huaan/huaan:[镜像版本号] 
# 讲镜像推送到仓库 
$ sudo docker push registry.cn-hangzhou.aliyuncs.com/huaan/huaan:[镜像版本号]
~~~

3.拉取镜像

~~~shell
docker pull registry.cn-hangzhou.aliyuncs.com/coldest7/mytom:v1
~~~



# 报错

### [解决报错WARNING: IPv4 forwarding is disabled. Networking will not work.](https://www.cnblogs.com/python-wen/p/11224828.html)

​	第一步：在宿主机上执行echo "net.ipv4.ip_forward=1" >>/usr/lib/sysctl.d/00-system.conf

​	第二步：重启network和docker服务

[root@localhost /]# systemctl restart network && systemctl restart docker

### Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

​	服务没启 service docker start。



## 安装ngnix

~~~shell
搜索镜像 docker search nginx
下载镜像 docker pull nginx
启动 docker run -d --name nginx01 -p 3344:80 nginx (-d:后台启动 --name：自定义名字 -p： 外网3344端口对应内部80端口 )
查看容器 docker ps
验证 curl localhost:3344
进入容器 docker exec -it nginx01 /bin/bash
查看配置文件 whereis nginx
退出容器 exit
停止  docker stop 687b8ecce7d8
~~~

# **实践**

## 安装tomcat

~~~shell
下载 docker pull Tomcat:9.0
~~~

## 部署es+kibana

~~~shell
# es 暴露的端口多 # es十分耗费内存 # es的数据需要挂载出去
--net somenetwork 网络配置

下载启动  docker run -d --name elasticsearch  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2

启动后linux会变卡，耗费大量内存。增加es内存限制（配置文件）
查看cpu状态 docker stats 
docker run -d --name elasticsearch02  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2

测试是否启动 curl localhost:9200

~~~

## 可视化portainer

~~~shell
docker图形化管理工具
docker run -d -p 8088:9000  --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
~~~

## commit镜像

~~~shell
自定义容器打包成镜像
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[tag]
docker images 查看
~~~

## 容器数据卷

~~~shell
docker容器中产生的数据同步到本地。（目录挂载）
docker run -v 主机目录:容器内目录
docker run -it -v /home/ceshi:/home centos /bin/bash
docker inspect dfe5502faaf6(容器id)
~~~

![image-20201215100411308](E:\dev\picture\image-20201215100411308.png)

![image-20201215100846488](E:\dev\picture\image-20201215100846488.png)

![image-20201215101411531](E:\dev\picture\image-20201215101411531.png)

### 实践：mysql数据持久化同步

~~~shell
获取镜像  docker pull mysql:5.7

安装mysql需要设置密码
官方：docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
本地测试：	docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

navicat测试连接

删除容器(数据依然存在) 	docker rm -f mysql01
~~~

### 具名挂载、匿名挂载

~~~shell
# 匿名挂载（不指定外部路径） -v 容器路径
docker run -d -P --name nginx01 -v /etc/nginx nginx
# 查看本地卷的情况   docker volume ls
DRIVER              VOLUME NAME
local            65541sdfd6555654sdfasdfc6522465221asdfdshdf

# 具名挂载 -v 卷名:容器路径  大多数情况使用具名挂载
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx
[root@aubin ~]# docker volume ls
DRIVER              VOLUME NAME
local               juming-nginx
# 查看卷信息
docker volunme inspect juming-nginx
~~~

![image-20201215143510088](E:\dev\picture\image-20201215143510088.png)

所有docker容器内的卷，没有指定目录的都是在	/var/lib/docker/volumes/* /_data下

~~~shell
# 如何确定是具名挂载还是匿名挂载，还是指定路径挂载
-v 容器路径       匿名挂载
-v 卷名:容器路径  具名挂载(卷名前不带/)
-v /宿主机路径:容器路径  指定路径挂载

启动的同时修改文件读写权限，ro:readonly只读，容器内无法修改，只能从外部修改。rw：readwrite可读写
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:rw nginx

~~~

## 初识Dockerfile

~~~shell
Dockerfile 就是用来构建docker镜像的构建文件！命令脚本
通过这个脚本可以生成镜像，镜像是一层一层的，脚本一个个的命令，每个命令都是一层

创建dockerfile文件（指令大写）
FROM centos
VOLUME ["volume01","volume02"] --》匿名挂载
CMD echo "----end---"
CMD /bin/bash

创建镜像
docker build -f dockerfile1 -t kuangshen/centos:1.0 .

查看 docker images

启动 docker run -it fba98a6ee3f1 /bin/bash
~~~

![image-20201215150641741](E:\dev\picture\image-20201215150641741.png)

这个卷和外部有一个同步的目录，查看容器详情   docker inspect d3b36ad9b201

![image-20201215150959564](E:\dev\picture\image-20201215150959564.png)

## 数据卷容器

多个mysql同步数据（多个容器同步）

![image-20201215151447685](E:\dev\picture\image-20201215151447685.png)

~~~shell
启动三个容器
~~~

![image-20201215151754306](E:\dev\picture\image-20201215151754306.png)

![image-20201215152241276](E:\dev\picture\image-20201215152241276.png)

![image-20201215152459780](E:\dev\picture\image-20201215152459780.png)

> 删除关联的容器，数据依旧存在。删除docker01对docker02和docker03的数据无影响。容器内执行删除文件操作，关联的容器中此文件都会删除。

## Dockerfile

构建步骤

1. 编写dockerfile文件

2. docker build 构建镜像

3. docker run 运行镜像

4. docker push 发布镜像

   官方：

   ![image-20201215153900820](E:\dev\picture\image-20201215153900820.png)

   ![image-20201215153920411](E:\dev\picture\image-20201215153920411.png)
   
### 测试：搭建centos

   1. 编写dockerfile文件
   
      ~~~shell
      [root@aubin dockerfile]# cat mydockerfile-centos 
      FROM centos
      MAINTAINER ks<email.qq.com>
      ENV MYPATH /usr/local
      WORKDIR $MYPATH
      
      RUN yum -y install vim
      RUN yum -y install net-tools
      
      EXPOSE 80
      
      CMD echo $MYPATH
      CMD echo "---end---"
      CMD /bin/bash
      ~~~
   
   2. 通过文件构建镜像
   
      ~~~shell
      docker build -f dockerfile文件路径 -t 镜像名:版本号 .
      # 成功
      Successfully built 35151329ca1c
      Successfully tagged mycentos:0.1
      ~~~
   
   3. 运行测试    docker run -it mycentos:0.1

### 实践：tomcat镜像

   1. 准备镜像文件：tomcat、jdk压缩包
   
      ![image-20201217142936663](E:\dev\picture\image-20201217142936663.png)
   
   2. 编写dockerfile文件，官方命名Dockerfile，build时会自动寻找此文件，不需要-f指定
   
      ~~~shell
      [root@aubin tomcat]# vim Dockerfile
      [root@aubin tomcat]# cat Dockerfile 
      FROM centos
      MAINTAINER wd<email>
      
      COPY readme.txt /usr/local/readme.txt
      ADD jdk-8u11-linux-x64.tar.gz /usr/local/
      ADD apache-tomcat-9.0.41.tar.gz /usr/local/
      
      RUN yum -y install vim
      ENV MYPATH /usr/local
      
      WORKDIR $MYPATH
      
      ENV JAVA_HOME /usr/local/jdk1.8.0_11
      ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
      ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.41
      ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.41
      ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
      
      EXPOSE 8080
      
      CMD /usr/local/apache-tomcat-9.0.41/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.41/bin/logs/catalina.out
      ~~~
   
   3. 构建镜像 执行  docker build -t 镜像名 .
   
   4. 启动镜像,暴露端口，挂载目录
   
      ~~~shell
      docker run -d -p 9090:8080 --name kstomcat -v /home/dockerfile/tomcat/test:/usr/local/apache-tomcat-9.0.41/webapps/test -v /home/dockerfile/tomcat/tomcatlogs/:/usr/local/apache-tomcat-9.0.41/logs diytomcat
      ~~~
   
   5. 访问9090端口，tomcat页面
   
   6. 外部宿主机test目录新建web.xml
   
      ![image-20201217154313864](E:\dev\picture\image-20201217154313864.png)
   
      ~~~shell
      
       <?xml version="1.0" encoding="UTF-8"?>
        <web-app xmlns="http://java.sun.com/xml/ns/javaee"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                                     http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
                 version="2.5">
      
        </web-app>
      ~~~
   
      ~~~jsp
      <%@ page language="java" contentType="text/html; charset=UTF-8"
          pageEncoding="UTF-8"%>
      <!DOCTYPE html>
      <html>
      <head>
      <meta charset="utf-8">
      <title>菜鸟教程(runoob.com)</title>
      </head>
      <body>
      <p>
         今天的日期是: <%= (new java.util.Date()).toLocaleString()%>
      </p>
      </body> 
      </html>
      ~~~
   
   7. 访问 http://192.168.75.128:9090/test/

## Docker网络

   1. 

   

   


