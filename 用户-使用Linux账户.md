# 注意事项
1.采用"使用Linux账户"的方式使用GPU服务器的情况下, 同一个导师的所有学生共同使用同一个账户. 第二步"安装Anaconda 更换Conda源"只需要一位学生在对应导师的账户里执行一次即可.

# 进入Ubuntu系统
1.进入 [Xshell官网](https://www.netsarang.com/zh/free-for-home-school/) 填上姓名与邮件, 下载免费的Xshell和Xftp.

2.在Xshell中输入分配到的IP地址, 采用默认22端口, 再输入导师的账号和密码, 即可进入Ubuntu账户.

# 安装Anaconda 更换Conda源
1.输入```wget -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-2021.11-Linux-x86_64.sh```,
下载完成后, 输入```chmod +x Anaconda3-2021.11-Linux-x86_64.sh```, 再输入```bash Anaconda3-2021.11-Linux-x86_64.sh```开始安装.

2.首先一直按着Enter阅读用户协议, 读完后输入yes即可. 输入完yes后, 询问你要不要改变安装路径, 这里不需要改变路径, 安装在/root文件夹即可. 按回车即可开始安装, 如下图:  
![img_1.png](img_1.png)  
安装完毕后, 询问你要不要进行conda初始化, 输入yes.  
![img_2.png](img_2.png)  
完成以后, 输入```source ~/.bashrc```命令, 此时应该就已经激活了base环境.

3.更换conda源为清华源: 输入```vim ~/.condarc```, 按i键进行编辑, 将[Anaconda 镜像使用帮助](https://mirror.tuna.tsinghua.edu.cn/help/anaconda/)页面中的源配置文件粘贴进去, 然后Esc再```:wq```保存并退出.退出后, 执行一下```conda clean -i```命令清除索引缓存.
