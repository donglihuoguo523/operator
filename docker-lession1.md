# 一、Linux Namespace

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

每个容器可以有不同的时间，而不是所有容器时间统一为宿主机的时间

# 二、使用apt/yum/二进制安装指定版本的docker

### rpm包下载地址:

- 官网下载地址：https://download.docker.com/linux/centos/7/x86_64/stable/Packages/
- 阿里镜像下载地址：https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/

### CentOS 系统离线使用rpm包部署docker

```shell
1、首先需要在联网机器上下载好docker需要到各种依赖包，前提是该机器之前没有安装过docker，否则可能会出现包下载不全，在不联网到机器无法顺利部署
mkdir $HOME/docker-packages
# 查看镜像仓库有哪些docker版本可通过命令查看：yum list docker-ce.x86_64 --showduplicates | sort -r
yum install --downloaddir=$HOME/docker-packages/ --downloadonly docker-ce-17.03.1.ce-1.el7.centos
2、打包下载好的镜像目录，传到不联网的机器上
cd $HOME  && tar -czvf docker-packages.tar.gz docker-packages
scp docker-packages.tar.gz user@IP:/$HOME/
3、登录不联网机器部署
cd $HOME && tar -zxvf docker-packages.tar.gz
cd docker-packages && yum localinstall *.rpm 
```

### Ubuntu系统离线使用deb包部署docker 

```shell
1、首先需要在联网机器上下载好docker需要到各种依赖包，前提是该机器之前没有安装过docker，否则可能会出现包下载不全，在不联网到机器无法顺利部署
# 查看镜像仓库有哪些docker版本可通过命令查看：apt-cache madison docker-ce
# Ubuntu下载好的dpkg包默认存放在/var/cache/apt/archives/ 目录下，所以需要提前清空该目录
rm -rf /var/cache/apt/archives/
apt-get install -d docker-ce=5:20.10.17~3-0~ubuntu-jammy
2、打包下载好的软件目录，传到不联网的机器上
cd /var/cache/apt/  && tar -czvf docker-packages.tar.gz archives
scp docker-packages.tar.gz user@IP:/$HOME/
3、登录不联网机器部署
cd $HOME && tar -zxvf docker-packages.tar.gz
cd archives && dpkg -i *.deb
```

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

# 三、docker基础命令

```shell
# docker run -it centos:7.9.2009 ## docker attach 63fbc2d5a3ec #attach进入容器后操作会在多窗口同步且退出后其它终端也会一起退出。
# docker build #从dockerfile 构建镜像
# docker commit -a "yuhao yuhao@haogedu.com" -m "v2" --change="EXPOSE 80 443" 8ddf4ce923ef new-nginx-image #将容器提交为一个本地镜像
# docker cp 源 目的 #从容器和宿主机相互拷贝文件或目录
# docker create -it --name test1 nginx:1.20.2 #创建一个新的容器且创建后的容器处于退出状态
# docker diff 8ddf4ce923ef #对比容器和镜像有差异的文件或目录
# docker exec -it 40e6379cf371 sh/bash #进入到容器执行命令操作，推荐使用此方式
# docker export 8ddf4ce923ef -o new-nginx-image.tar.gz #将容器的文件系统导出为一个本地压缩包,非镜像格式
# docker history centos:7.9.2009 #查看的镜像的构建历史
# docker images #查看本地所有镜像
# docker info #显示系统信息
# docker inspect 50fe74b50e0d #显示docker对象(镜像、网络、容器等)的详细信息
# docker kill $(docker ps -a -q) 强制关闭所有运行中的容器
# docker load -i nginx-1.20.2.tar #从一个tar包或标准输入导入镜像
# docker login #登录镜像仓库
# docker logout #登出镜像仓库
# docker logs -f nginx-container-test1 #持续查看容器标准输出和错误输出的日志
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
# docker save 50fe74b50e0d > nginx-1.20.2.tar #保存一个或多个镜像到一个压缩文件(默认是标准输出)
# docker start ID/容器名称 #启动一个或多个容器
# docker stats #显示容器资源的实时统计信息
# docker stop ID/容器名称 #停止一个或多个容器
# docker tag nginx:1.20.2 harbor.magedu.net/myserver/nginx:1.20.2 #为镜像添加一个新的tag
# docker update 容器 --cpus 2 #更新容器配置信息，比如资源限制的值
# docker version #显示docker client和docker server的版本信息
```

# 四、Docker 数据卷Volume

### Docker数据管理

如果正在运行中的容器修如果生成了新的数据或者修改了现有的一个已经存在的文件内容，那么新产生的数据将会被复制到读写层进行持久化保存，这个读写层也就是容器的工作目录，此即“写时复制(COW) copy on write”机制。

通过 docker inspect <containerid> 命令可以看到数据的各个层级， 各层作用如下:

- Lower Dir：image镜像层(镜像本身，只读)

- Upper Dir：容器的上层(读写)

-  Merged Dir：容器的文件系统，使用Union FS（联合文件系统）将lowerdir和upper Dir：合并给容器使用。

- Work Dir：容器在 宿主机的工作目录


Docker的镜像是分层设计的，镜像层是只读的，通过镜像启动的容器添加了一层可读写的文件系统，用户写入的数据都保存在这一层当中。

如果要将写入到容器的数据永久保存，则需要将容器中的数据保存到宿主机的指定目录，目前Docker的数据类型分为两种:

- 一是数据卷(data volume)，数据卷实际上就是宿主机上的目录或者是文件，可以被直接mount到容器当中作为容器的本地文件系统使

用，实际生产环境中，需要针对不同类型的服务、不同类型的数据存储要求做相应的规划，最终保证服务的可扩展性、稳定性以及数据的

安全性。基于数据卷通过将宿主机的文件或目录挂载到容器的指定目录，当容器中的挂载目录产生的新数据即可间接的保存到宿主机以实

现持久化的目的. 

- 二是数据卷容器(Data volume container), 数据卷容器是将宿主机的目录挂载至一个专用的数据卷容器，然后让其他容器通过数据卷容器

继承挂载的目录或文件，以实现将新产生的数据持久化到宿主机的目的。



### Docker数据管理-docker创建数据卷

```shell
创建数据卷
# docker volume create nginx-data
# docker volume ls
DRIVER VOLUME NAME
local nginx-data
启动容器并挂载至本地数据卷
# docker run -it -d -p 80:80 -v nginx-data:/data nginx:1.23.2
4445e4640c74e1c503a7f8c704270abd58c5bc70aacf600e39945f7fd74ac810
进入容器，并创建index.html文件
# docker exec -it 4445e4640c74 bash
root@4445e4640c74:/# echo "nginx web" > /data/index.html
root@4445e4640c74:/# exit
在宿主机上可以看到文件被创建出来
# ls /var/lib/docker/volumes/nginx-data/_data/index.html
/var/lib/docker/volumes/nginx-data/_data/index.html
```

### Docker 数据管理-数据目录挂载

```shell
1、宿主机创建目录
# mkdir /data/testapp –p
2、宿主机创建文件
# echo "testapp web page" > /data/testapp/index.html
3、启动第一个Nginx容器
# docker run -d --name web1 -v /data/testapp:/usr/share/nginx/html/testapp -p 80:80 nginx:1.23.2
4、启动第二个Nginx容器
# docker run -d --name web2 -v /data/testapp:/usr/share/nginx/html/testapp:ro -p 81:80 nginx:1.23.2
5、可以通过浏览器访问者两个容器服务：http://IP:80/testapp,http://IP:81/testapp,会看到两个服务的页面显示内容完全一致

```

### Docker 数据管理-数据目录及配置多卷挂载

```shell
nginx多卷挂载：
1、创建Nginx目录
# mkdir /data/nginx/conf -p
2、复制刚创建的web1容器内的nginx配置至宿主机目录下
# docker cp web1:/etc/nginx/nginx.conf /data/nginx/conf/
3、运行第三个Nginx容器，可以看到可以同时挂载多个数据卷
# docker run -d --name web3 -v /data/testapp:/usr/share/nginx/html/testapp -v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf:ro -p 83:80 nginx:1.23.2
mysql容器示例：
# docker pull mysql:5.7.38
# docker run -it -d -p 3306:3306 -v /data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7.38

```

### Docker 数据管理-删除容器

- 创建容器的时候指定参数会删除/var/lib/docker/containers/的容器数据目录，但是不会删除数据卷的内容，如下

  ```shell
  # docker rm -f web3
  # ls /data/testapp/index.html
  /data/testapp/index.html #挂载的数据卷不会被删除
  ```

- 数据卷的特点及使用
  - 数据卷是宿主机的目录或者文件，并且可以在多个容器之间共同使用。
  - 在宿主机对数据卷更改数据后会在所有容器里面会立即更新。
  - 数据卷的数据可以持久保存，即使删除使用使用该容器卷的容器也不影响。
  - 在容器里面的写入数据不会影响到镜像本身。

- 数据卷使用场景
  - 容器数据持久化(mysql数据、nginx日志等类型)
  - 静态web页面挂载
  - 应用配置文件挂载
  - 多容器间的目录或文件共享

### Docker 数据管理-数据卷容器

数据卷容器功能是可以让数据在多个docker容器之间共享，即先要创建一个后台运行的A容器作为Server，之后创建的B容器、C容

器等都可以同时访问A容器的内容，因此数据卷容器用于为其它容器提供卷的挂载继承服务，数据卷为其它容器提供数据读写服务，

A容器称为server端、其它容器成为client端：

#### 卷容器Server

```shell
# docker run -d --name volume-server -v /data/testapp:/usr/share/nginx/html/testapp -v /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf:ro registry.k8s.io/pause:3.8
```

#### 卷容器client

```shell
client1
# docker run -d --name web1 --volumes-from volume-server -p80:80 nginx:1.23.2
client2
# docker run -d --name web2 --volumes-from volume-server -p81:80 nginx:1.23.2
```

#### 特点

- 适用于同类服务的数据卷共享

- client会继承卷server挂载和挂载权限

- 停止卷server，也不影响已经运行的容器、甚至也不影响新建容器

- 删除卷server，不影响已经运行的容器，但是不能新建容器

# 容器网路

### Docker网络

Docker服务安装完成之后，默认在每个宿主机会生成一个名称为docker0的网卡其IP地址都是172.17.0.1/16，并且会生成三种不能类型的网络，如下：

```shell
# ifconfig docker0
另外会额外创建三个默认网络，用于不同的使用场景：
# docker network list
NETWORK ID		NAME	DRIVER	SCOPE
438a9be14ef8 	bridge  bridge	local 	#桥接网络，默认使用的模式，容器基于SNAT进行地址转换访问宿主机以外的环境
4c026356e4d1	host 	host	local 	#host网络，直接使用宿主机的网络( 不创建net namespace)，性能最好，但是容器端口不能冲突
8d70da095b8e	none 	null	local 	#空网络，容器不会分配有效的IP地址(只有一个回环网卡用于内部通信)，用于离线数据处理等场景。
```

### Docker网络-bridge模式

docker的默认模式即不指定任何模式就是bridge模式，也是目前使用比较多的网络模式，此模式创建的容器会为每一个容器分配自己的网络IP等信息，并将容器连接到一个虚拟网桥与外界通信。

```shell
# docker network inspect bridge
# docker run -it -d --name nginx-web1-bridge-test-container -p 80:80 --net=bridge nginx:1.23.2
```

### Docker网络-host模式

host模式就是直接使用宿主机的IP地址，创建的容器如果指定了使用host模式，那么新创建的容器不会创建自己的虚拟网卡，而是直接使用宿主机的网卡和IP地址，因此在容器里面查看到的IP信息就是宿主机的信息，访问容器的时候直接使用宿主机IP+容器端口即可，而且某一个端口一旦被其中一个容器占用那么其它容器就不能再监听此端口，不过容器的其它资源比如容器的文件系统、容器系统进程等还是基于namespace相互隔离。

**此模式的网络性能最高，但是各容器之间端口不能相同，适用于运行容器端口比较固定的业务**

```shell
# docker run -it -d --name nginx-web1-host-test-container -p 80:80 --net=host nginx:1.23.2
```

### Docker网络-none模式

none模式就是无IP模式，在使用none 模式后，docker 容器不会进行任何网络配置，其没有网卡、没有IP也没有路由，因此默认无法与外界通信，需要手动添加网卡配置IP等，所以极少使用。

```shell
# docker run -it -d --name nginx-web1-none-test-container -p 80:80 --net=none busybox sleep 10000000
# docker exec -it 3c54e90c209a sh
# ifconfig
lo Link encap:Local Loopback
inet addr:127.0.0.1 Mask:255.0.0.0
UP LOOPBACK RUNNING MTU:65536 Metric:1
RX packets:0 errors:0 dropped:0 overruns:0 frame:0
TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:0 (0.0 B) TX bytes:0 (0.0 B)
# ping 223.6.6.6
PING 223.6.6.6 (223.6.6.6): 56 data bytes
ping: sendto: Network is unreachable
```

### Docker网络-container模式

Container模式即容器模式，使用参数: **--net container:目标容器名称/ID** 指定,使用此模式创建的容器需指定和一个已经存在的容器共享一个网络namespace，而不会创建独立的namespace，即新创建的容器不会创建自己的网卡也不会配置自己的IP，而是和一个已经存在的被指定的目标容器共享对方的IP和端口范围，因此这个容器的端口不能和被指定的目标容器端口冲突，除了网络之外的文件系统、用户信息、进程信息等仍然保持相互隔离，两个容器的进程可以通过lo网卡及容器IP进行通信。

```shell
# docker run -it -d --name nginx-container -p 80:80 --net=bridge nginx:1.22.0-alpine
# docker run -it -d --name php-container --net container:nginx-container php:7.4.30-fpm-alpine
```

