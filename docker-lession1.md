# Linux Namespace

| 隔离类型 | 功能 | 系统调用参数 | 内核版本 |
| -------- | ---- | ------------ | -------- |
| MNT Namespace(mount) | 提供磁盘挂载点和文件系统的隔离能力 | CLONE_NEWNS | Linux 2.4.19 |
| IPC Namespace(Inter-Process Communication) | 提供进程间通信的隔离能力 | CLONE_NEWIPC | Linux 2.6.19 |
| UTS Namespace(UNIX Timesharing System) | 提供主机名隔离能力 | CLONE_NEWUTS | Linux 2.6.19 |
| PID Namespace(Process Identification) | 提供进程隔离能力 | CLONE_NEWPID | Linux 2.6.24 |
| Net Namespace(network) | 提供网络隔离能力 | CLONE_NEWNET | Linux 2.6.29 |
| User Namespace(user) | 提供用户隔离能力 | CLONE_NEWUSER | Linux 3.8 |
| Time Namespace | 提供时间隔离能力 | CLONE_NEWTIME | Linux 5.6 |
| Syslog Namespace | 提供syslog隔离能力 |  |  |
| Control group(cgroup) Namespace | 提供进程所属的控制组到身份隔离 |  | Linux 4.6 |

# 使用apt/yum/二进制安装指定版本的docker

### MNT Namespace

类似 chroot，将一个进程放到一个特定的目录执行。mnt 命名空间允许不同命名空间的进程看到的文件结构不同，这样每个命名空间 中的进程所看到的文件目录就被隔离开了。同 chroot 不同，每个命名空间中的容器在 /proc/mounts 的信息只包含所在命名空间的 mount point。

### IPC Namespace

容器中进程交互还是采用了 Linux 常见的进程间交互方法(interprocess communication – IPC), 包括信号量、消息队列和共享内存等。然而同 VM 不同的是，容器的进程间交互实际上还是 host 上具有相同 pid 命名空间中的进程间交互，因此需要在 IPC 资源申请时加入命名空间信息，每个 IPC 资源有一个唯一的 32 位 id。

### UTS Namespace

UTS(“UNIX Time-sharing System”) 命名空间允许每个容器拥有独立的 hostname 和 domain name, 使其在网络上可以被视作一个独立的节点而非 主机上的一个进程。

### PID Namespace

不同用户的进程就是通过 pid 命名空间隔离开的，且不同命名空间中可以有相同 pid。在同一个Namespace中只能看到当前命名空间的进程。所有的 LXC 进程在 Docker 中的父进程为Docker进程，每个 LXC 进程具有不同的命名空间。同时由于允许嵌套，因此可以很方便的实现嵌套的 Docker 容器。

### Net Namespace

有了 pid 命名空间, 每个命名空间中的 pid 能够相互隔离，但是网络端口还是共享 host 的端口。网络隔离是通过 net 命名空间实现的， 每个 net 命名空间有独立的 网络设备, IP 地址, 路由表, /proc/net 目录。这样每个容器的网络就能隔离开来。Docker 默认采用 veth 的方式，将容器中的虚拟网卡同 host 上的一 个Docker 网桥 docker0 连接在一起。

### User Namespace

每个容器可以有不同的用户和组 id, 也就是说可以在容器内用容器内部的用户执行程序而非主机上的用户。

### Time Namespace



# 使用apt/yum/二进制安装指定版本的docker

### rpm包下载地址:

- 官网下载地址：https://download.docker.com/linux/centos/7/x86_64/stable/Packages/

- 阿里镜像下载地址：https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/

### 二进制安装包：

- 清华大学镜像源： https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/static/stable/x86_64/

### apt/yum 在线安装：

- 参考清华大学镜像源：https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/

- 参考阿里云镜像源：https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.3e221b11xaRalQ

### CentOS 部署Docker

```shell
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3
sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
# Step 4: 更新并安装Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start

# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，您可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将[docker-ce-test]下方的enabled=0修改为enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```

### Ubuntu 部署Docker

```shell
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce

# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]
```

# docker基础命令

### docker命令详解：

```shell
# docker run -it centos:7.9.2009 ## docker attach 63fbc2d5a3ec #attach进入容器后操作会在多窗口同步且退出后其它终端也会一起退出。
# docker build #从dockerfile 构建镜像
# docker commit -a "jack jack@magedu.com" -m "v2" --change="EXPOSE 80 443" 8ddf4ce923ef new-nginx-image #将容器提交为一个本地镜像
# docker cp 源 目的 #从容器和宿主机相互拷贝文件或目录
# docker create -it --name test1 nginx:1.20.2 #创建一个新的容器且创建后的容器处于退出状态
# docker diff 8ddf4ce923ef #对比容器和镜像有差异的文件或目录
# docker events #获取dockerd的实时事件,创建删除容器等操作
# docker exec -it 40e6379cf371 sh/bash #推进入到容器执行命令操作，推荐使用此方式
# docker export 8ddf4ce923ef -o new-nginx-image.tar.gz #将容器的文件系统导出为一个本地压缩包,非镜像格式
# docker history centos:7.9.2009 #查看的镜像的构建历史
# docker images #查看本地所有镜像
# docker import new-nginx-image.tar.gz #导入export导出的压缩包，导入后的镜像不完整，不能用于创建容器
# docker info #显示系统信息
# docker inspect 50fe74b50e0d #显示docker对象(镜像、网络、容器等)的详细信息
# docker kill $(docker ps -a -q) 强制关闭所有运行中的容器
# docker load -i nginx-1.20.2.tar.gz #从一个tar包或标准输入导入镜像
# docker login #登录镜像仓库
# docker logout #登出镜像仓库
# docker logs -f nginx-container-test1 #持续查看容器标准输出和错误输出的日志
# docker pause 81b344cff55d #暂停一个或者多个容器
# docker port 81b344cff55d #列出一个容器端口映射关系
# docker ps #列出容器,加上-a是列出包含为运行的所有容器
# docker pull nginx:1.20.2 #从镜像仓库下载镜像
# docker push nginx:1.20.2 #从本地上传镜像到镜像仓库(需要登录认证)
# docker rename awesome_cerf nginx-container1 #重命名容器
# docker restart ID/容器名称 #重启容器
# docker rm -f 11445b3a84d3 #强制删除运行中的容器
# docker rm -f `docker ps -aq -f status=exited` #批量删除已退出容器
# docker rm -f $(docker ps -a -q) #批量删除所有容器
# docker rmi -f 53ec353d8dc4 90a4cd9dfe4c #删除一个或多个镜像
# docker run -it docker.io/centos bash #创建并进入容器，ctrl+p+q退出容器不注销
# docker run -it --name nginx-test1 nginx:1.20.2 #自定义容器名称
# docker run -p 80:80/tcp -p 443:443/tcp -p 53:53/udp --name nginx-container-test1 nginx:1.20.2 #创建容器并指定多端口映射
# docker run -d -p 80:80 --name nginx-container-test1 nginx:1.20.2 #后台运行容器
# docker run -it --rm --name nginx-delete-test nginx:1.20.2 bash #单次运行容器
# docker run -it -d centos:7.9.2009 /usr/bin/tail -f '/etc/hosts' #创建容器的时候传递命令及参数
# docker save 50fe74b50e0d > nginx-1.20.2.tar.gz #保存一个或多个镜像到一个压缩文件(默认是标准输出)
# docker search nginx #搜索镜像
# docker start ID/容器名称 #启动一个或多个容器
# docker stats #显示容器资源的实时统计信息
# docker stop ID/容器名称 #停止一个或多个容器
# docker tag nginx:1.20.2 harbor.magedu.net/myserver/nginx:1.20.2 #为镜像添加一个新的tag
# docker unpause 81b344cff55d #取消一个或多个容器的暂停
# docker update 容器 --cpus 2 #更新容器配置信息，比如资源限制的值
# docker version #显示docker client和docker server的版本信息
# docker wait #一直等待容器退出并显示容器的退出状态码
```
