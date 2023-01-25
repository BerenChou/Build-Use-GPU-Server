TO DO

# 需求
1.GPU计算资源能够多用户共享, 同时每个用户的开发环境能够相互隔离, 且用户的任何操作都只限于自己被分配到的环境的内部, 无法干扰宿主机.

2.可支持用户对开发环境的不同需求, 如指定CUDA版本或者无CUDA. 且管理员可根据用户需求, 使用几行命令迅速构建开发环境并进行分发.

3.管理员易于维护. 可以迅速新建用户开发环境, 或者停止, 重启, 删除.

# 解决方案
1.宿主机安装Ubuntu与Docker, 管理员使用Docker创建一个或多个镜像(类比面向对象编程的class), 每个镜像可以有不同的开发环境, 比如A镜像CUDA版本为11.4, B镜像CUDA版本为10.1.
基于镜像, 可以创建容器(类比面向对象编程的object, object是从class中派生得来), 每个容器对应有一个IP地址与端口, 将其分发给用户, 用户即可使用Xshell等软件, 使用ssh连接到容器中, 使用GPU资源.

# 对Ubuntu系统的调整
1.设置一下root账户密码: ```sudo passwd root```.

2.为了避免每次sudo都要输入密码, 这里配置一下visudo: ```sudo visudo```, 在文件最后加上一句```xxx(改为自己账户名) ALL=(ALL) NOPASSWD: ALL```, Ctrl+o键写入修改再回车保存, 再Ctrl+x键退出.

3.先apt-get install vim, 修改Ubuntu镜像源为阿里源: ```sudo vim /etc/apt/sources.list```, 全部删除后, 写入[阿里云Ubuntu 镜像](https://developer.aliyun.com/mirror/ubuntu)中关于Ubuntu 20.04(focal)的源. 然后执行```sudo apt update```和```sudo apt upgrade```命令.

4.安装ssh: ```sudo apt install ssh```, 安装net-tools: ```sudo apt install net-tools```, 安装xrdp```sudo apt install xrdp```, 安装完成xrdp, 服务将会自动启动，可以输入下面的命令验证它```sudo systemctl status xrdp```,
默认情况下, xrdp使用/etc/ssl/private/ssl-cert-snakeoil.key, 它仅仅对ssl-cert用户组成语可读, 所以需要运行下面的命令, 将xrdp用户添加到这个用户组: ```sudo adduser xrdp ssl-cert```, ```sudo systemctl restart xrdp```.

5.重启电脑, 一定要重启!

6.打开Ubuntu系统中的软件和更新, 找到标签栏中的附加驱动, 点进去以后, 选择第一个驱动, 有"使用NVIDIA driver metapackage来自nvidia-driver-xxx(专有, tested)"字样的, 再点击应用更改, 即可开始安装. 安装完毕后, 重启电脑. 重启完毕后再输入```sudo apt update```和```sudo apt upgrade```更新一下软件, 然后再次重启.

# 安装Docker和Nvidia Container Toolkit
1.安全按照[Docker官方文档](https://docs.docker.com/engine/install/ubuntu/#installation-methods)中的Install using the repository方法来安装.
2.完全按照[Nvidai Container Toolkit官方文档](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#installing-on-ubuntu-and-debian)中的方法来安装, 唯一需要注意的点是, 这句话:
![image](https://user-images.githubusercontent.com/46392714/143682609-6e27caae-2a65-4a46-914c-9dd60b2a7be7.png)
是这样执行的: 首先```sudo vim /etc/docker/daemon.json```, 然后加上一句话```"default-runtime": "nvidia",```, 如下图所示, 然后再按上图将Docker守护程序重启, 再按照文档执行后续步骤.
******

# 禁止Linux内核更新与更换Docker路径
1.先```uname -a```查看系统内核, 以笔者电脑举例, 系统内核为5.11.0-41-generic, 再```sudo dpkg --get-selections | grep linux```, 能看到linux-headers-5.11.0-41-generic和linux-image-5.11.0-41-generic和linux-modules-5.11.0-41-generic和linux-modules-extra-5.11.0-41-generic, 使用```sudo apt-mark hold```将上述4个包设定为不更新:
```
sudo apt-mark hold linux-headers-5.11.0-41-generic
sudo apt-mark hold linux-image-5.11.0-41-generic
sudo apt-mark hold linux-modules-5.11.0-41-generic
sudo apt-mark hold linux-modules-extra-5.11.0-41-generic
```

2.我们将固态硬盘挂载到了/目录, 将机械硬盘挂载到了/home目录, 而docker默认安装目录是在/var/lib/docker中, 随着Docker的使用, 固态硬盘有可能会空间不足, 所以这里笔者将Docker目录移动到了/home中的用户文件夹, 具体移动方法, 请参考[IBM Relocating the Docker Root directory](https://www.ibm.com/docs/en/z-logdata-analytics/5.1.0?topic=compose-relocating-docker-root-directory).

# 使用Dockerfile定制镜像
```mkdir DockerfileContext```, ```cd DockerfileContext```, ```touch Dockerfile```, ```vim Dockerfile```, 写入下面的内容:
```
FROM nvidia/cuda:11.3.0-cudnn8-devel-ubuntu20.04
MAINTAINER BerenChow<weixin:*******>
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone \
    && echo "root:root" | chpasswd \
    && apt-get update && apt-get install -y \
        vim \
        openssh-server \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean \
    && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config

ENTRYPOINT service ssh start && bash
```
这个Dockerfile文件基于NVIDIA官方的nvidia/cuda:11.3.0-cudnn8-devel-ubuntu20.04镜像, 加入了ssh功能, 使得用户端能通过ssh进入宿主机中的容器.

然后```sudo docker build -t nvidia/cuda:ssh-cuda11.3.0-cudnn8-ubuntu20.04 .```, 即可成功构建镜像, 并返回其镜像ID.

到此为止, 基于Ubuntu + Docker的远程GPU服务器构建完毕, 管理员请转向本文档项目的 管理员_创建容器.md 文件.
