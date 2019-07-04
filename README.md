#### Docker手册简版

由发行书籍：每天5分钟玩转docker容器技术 整理而来，写在图中发现，已经有一个Gitbook有相关的著作.


[https://yeasy.gitbooks.io/docker_practice/install/ubuntu.html#%E7%B3%BB%E7%BB%9F%E8%A6%81%E6%B1%82](https://yeasy.gitbooks.io/docker_practice/install/ubuntu.html#%E7%B3%BB%E7%BB%9F%E8%A6%81%E6%B1%82 "https://yeasy.gitbooks.io/docker_practice/install/ubuntu.html#%E7%B3%BB%E7%BB%9F%E8%A6%81%E6%B1%82")

- 安装忽略.
- 常用命令行.

```bash
# 拉取镜像
docker pull tensorflow/tensorflow:latest-gpu-py3

# 可以拉取NVIDIA官方镜像试试，运行镜像.client_port:host_port
docker run -it -d -p 80:80 --name=my tensorflow/tensorflow:latest-gpu-py3

# -v可以添加目录映射.

# 停止容器
docker stop/kill my

# 启动容器
docker start my

# 进入到容器，or可以第二行，创建新的terminal，所以exit不会关闭容器.
docker exec -it my bash

# exit退出，容器则关闭，连tensorflow的大图标都不显示.
# 进入到的是源terminal，于是exit就会关闭容器.
# 退出容器的方式是:ctrl+p,ctrl+q.
docker attach my
```
- 构建镜像

```bash
# 容器保存修改后的容器为镜像.
docker commit my my_add_new_thing

构建镜像的方式，以上的并不推荐，因为容易出错，效率低。

# 推荐使用Dockerfile文件创建，但是不建议自己创建镜像，而是使用官方镜像.
```
- 设置自动重启容器

```bash
# 个人测试貌似不太成功
# 无论何种情况关闭容器，都会自动重启容器.
docker run -d --restart=always image_name

# 启动进程退出代码非0，重启容器，最多三次.
docker run -d --restart=on-failture:3 
```
- 容器暂停

```bash
# 暂停/取消暂停.
docker pause
docker unpause
```
- 批量删除退出的容器

```bash
docker rm -v $(docker ps -aq -f status=exited)
```

```bash
# 内存，swap均分配200.
docker run -it -m 200M ubuntu

# 内存分配不一致，内存=200，swap=100.
docker run -it -m 200M --memory-swap=300M ubuntu

# cpu限额,cpu=1是逻辑核心4还是CPU个数只有一个.
docker run -it --name=myname -c 512 tensorflow/tensorflow:latest-gpu-py3 --cpu=1

# BlockIO带宽限额: bps与iops

# 限制容器写 dev/sda的速率为30MB/s
docker run -it --device-write-bps /dev/sda:30MB ubuntu
# 提示上述的代码描述并不是完整的，絮叨oflag=direct，具体使用可以到时候google，只要先知道有该功能即可.


# 使用网络：none, host(性能好，但是host使用过的端口不可以使用.), bridge.
docker run -it --network=none busybox

# 查看网桥信息，获得网桥也就是docker0/容器子网的信息.
brctl show
docker network inspect bridge

# 也就说网桥bridge配置的docker0的信息就是.
ifconfig docker0

"""
网络的拓扑结构,容器是.0/16分的更细，这里16表示掩码的位数是16位.
{httpd:(eth0:172.17.0.2)--(veth2857d)-->docker0:(172.17.0.1)}：DockerHost
"""
```

### user-defined网络.

host网络结构查看 `brctl show`，或者可以理解为网桥查看。

docker提供三种user-defined网络驱动：bridge、overlay、macvlan。后两种主要用于跨机网络...

##### 该部分比较复杂单独说明

- 方法一：bridge驱动创建bridge网络，不指定ip网段，由docker自动分配网段.

名称会是这样的: `virbr0` .

```bash
# bridge驱动创建bridge网络.
docker network create --driver bridge my_network
docker network inspect my_network # 网络会自动分配.

# 确定网络结构变化，会显示docker0和br-4fasgerlqj(自己创建的.)
brctl show

# 查看my_network 配置信息.
docker network inspect my_network
```
- 方法二：上述方法添加网段，带网段的创建.

名称会是这样的: `br-geaf5sf5 0` .

```bash
# 带网段的创建.
docker network create --driver bridge --subnet 172.22.16.0/24 --gateway 172.22.16.1 my_network_2

# 确定网络结构变化，会显示docker0和br-4fasgerlqj(自己创建的.)
brctl show # 网桥显示.

# 网关my_network_2在网桥br-4fasgerlqj上. 
ifconfig br-4fasgerlqj

# 查看my_network_2配置信息.
docker network inspect my_network_2
```

- 新网路的使用.

