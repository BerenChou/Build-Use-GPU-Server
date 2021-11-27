# 对Ubuntu系统的调整
1.设置一下root账户密码: ```sudo passwd root```.

2.为了避免每次sudo都要输入密码, 这里配置一下visudo: ```sudo visudo```, 在文件最后加上一句```xxx(改为自己账户名) ALL=(ALL) NOPASSWD: ALL```, Ctrl+o键写入修改再回车保存, 再Ctrl+x键退出.

3.修改Ubuntu镜像源为阿里源: ```sudo vim /etc/apt/sources.list```, 全部删除后, 写入[阿里云Ubuntu 镜像](https://developer.aliyun.com/mirror/ubuntu)中关于Ubuntu 20.04(focal)的源. 然后执行```sudo apt update```和```sudo apt upgrade```命令.

4.安装ssh: ```sudo apt install ssh```, 安装xrdp```sudo apt install xrdp```, 安装完成xrdp, 服务将会自动启动，可以输入下面的命令验证它```sudo systemctl status xrdp```,
默认情况下, xrdp使用/etc/ssl/private/ssl-cert-snakeoil.key, 它仅仅对ssl-cert用户组成语可读, 所以需要运行下面的命令, 将xrdp用户添加到这个用户组: ```sudo adduser xrdp ssl-cert```, ```sudo systemctl restart xrdp```.

5.
