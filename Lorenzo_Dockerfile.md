## Dockerfile
```
FROM nvidia/cuda:11.3.0-cudnn8-devel-ubuntu20.04
MAINTAINER zhouyunlai<weixin:leopraden>
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
