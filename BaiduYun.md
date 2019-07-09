[TOC]

# 百度云服务器（BCC）使用学习手记

### 缘起

由于工作所带来的数据处理方面的需求和兴趣，初步接触了linux、python、mysql等，也通过VMware虚拟机在个人华为笔记本上搭建了学习环境。然而在笔记本上用虚拟机，占用资源还是其次，每次启动实在麻烦，也不可能随时都带着笔记本，于是对云服务器产生了兴趣。碰巧看到百度最初级配置的云服务器价格109元/年，不知道是否划算，至少对比起来比同期的阿里云、腾讯元便宜，于是果断入手。反正也是为了学习瞎折腾，且试用一年再说。

![笔记本上搭建的学习环境](D:\Document\BaiduYun\BaiduYun\capture_20190708223440530.bmp)

[^图1]: 个人笔记本电脑上虚拟机搭建的学习环境

### 初识

#### 百度云服务器配置

| CPU  | 内存 | 带宽 | 系统盘 | 系统             | 价格     |
| ---- | ---- | ---- | ------ | ---------------- | -------- |
| 1核  | 1G   | 1M   | 40G    | Ubuntu 16.04 LTS | 109元/年 |

> CPU信息

```shell
(base) snail@instance-t009xqlo:~$ cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
1  Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz
```

> 系统信息

```shell
(base) snail@instance-t009xqlo:~$ cat /etc/issue
Ubuntu 16.04.1 LTS \n \l
```

> 网速测试

```shell
(base) snail@instance-t009xqlo:~$ speedtest-cli
Retrieving speedtest.net configuration...
Retrieving speedtest.net server list...
Testing from CNISP-Union Technology (Beijing) Co. (106.12.61.246)...
Selecting best server based on latency...
Hosted by China Telecom ZheJiang Branch (Hangzhou) [124.16 km]: 16.828 ms
Testing download speed........................................
Download: 61.41 Mbit/s
Testing upload speed..................................................
Upload: 1.04 Mbit/s
```

#### 百度云管理控制台

登陆百度云（https://cloud.baidu.com）后，便可以进入百度云的控制台，查看云服务器BCC实例信息，重启、停止实例，更改实例配置，重置系统等等。

PS：云服务器重置系统非常方便，最适合我这样瞎折腾的

> 云服务器BCC实例详情

![](D:\Document\BaiduYun\BaiduYun\capture_20190708231214160.bmp)

> 云服务器BCC实例监控

![](D:\Document\BaiduYun\BaiduYun\capture_20190708231621240.bmp)

> 百度云网页版VNC控制台

![](D:\Document\BaiduYun\BaiduYun\capture_20190708231931623.bmp)

#### 安装图形化界面

百度云服务器的Ubuntu系统是没有图形化界面的，安装很简单，在VNC控制台中执行命令`apt-get -y install x-window-system-core gnome-core gdm`安装gnome桌面环境后，执行命令`startx`即可启动图形化桌面效果。目前用处不大，略过。

#### putty使用SSH密钥登录实例

网页版的VNC控制台看看就好，用起来很不舒服，所以还是抓紧配置好putty。

- **下载putty**

- **在百度云服务器控制台中创建密钥对，并下载至本地**

- **运行puttygen.exe，载入所下载的密钥文件，生成私钥文件并保存至本地**

![](D:\Document\BaiduYun\BaiduYun\capture_20190708235128418.bmp)

- **运行putty.exe，在会话中输入百度云服务器实例的公网IP，并保存**

![](D:\Document\BaiduYun\BaiduYun\capture_20190708235745980.bmp)

- **在连接-SSH-认证中，打开此前生成的私钥文件**

![](D:\Document\BaiduYun\BaiduYun\capture_20190709000217866.bmp)

- **完成上述配置后，点击打开，即可通过putty远程连接百度云服务器实例**

------



### 搭建

下面开始在百度云服务器BCC实例（以下简称“服务器”）中瞎折腾地搭建学习环境

#### 在服务器中创建新用户

服务器在创建后初始用户为管理员root，但一直用root账号折腾似乎不太好，因此需要先创建一个新用户。

Ubuntu系统中创建新用户有两个命令：`useradd`和`adduser`。二者的区别是：`useradd`命令创建用户后，不会在/home目录下创建同名文件夹，也不会创建密码，因此想要正常登陆还需要后续设置，比较麻烦；而`adduser`命令创建用户后，会通过提示的形式完成相关所需操作，较为方便。

 PS：删除用户，只需使用命令`deluser 用户名`即可，一般希望把它留在系统上的文件也删除掉，此时可以使用`deluser -r 用户名`来实现。 

> 通过`adduser`命令创建新用户“newuser”

```shell
root@instance-t009xqlo:~# adduser newuser
Adding user `newuser' ...
Adding new group `newuser' (1003) ...
Adding new user `newuser' (1003) with group `newuser' ...
Creating home directory `/home/newuser' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for newuser
Enter the new value, or press ENTER for the default
        Full Name []: newuser
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
root@instance-t009xqlo:~#
```

但此时新用户“newuser”执行`sudo`命令会出现提示“newuser is not in the sudoers file.  This incident will be reported”，还需要将新用户添加到sudoers列表中。

> 将新用户添加到sudoers列表中

```shell
#进入管理员root权限
newuser@instance-t009xqlo:~$ su
Password:
#用时vim编辑sudoers列表
root@instance-t009xqlo:/home/newuser# vim /etc/sudoers
```

![](D:\Document\BaiduYun\BaiduYun\capture_20190709004908720.bmp)

在sudoers列表中添加上面一行，随后强制保存退出（`wq!`）即可。

#### 在服务器中创建FTP服务

为了便于在服务器和本地电脑间实现安全的文件传输，在服务器上搭建FTP服务。该部分配置主要参见了CSDN博文（https://blog.csdn.net/soslinken/article/details/79304076）。

- **安装vsftpd**

  ```shell
  $ sudo apt-get install vsftpd
  ```

- **备份vsftpd配置文件**

  ```shell
  $ sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
  ```

- **修改vsftpd配置文件**

  ```shell
  $ sudo vim /etc/vsftpd.conf
  ```

  配置文件主要配置如下

  ```shell
  listen=NO
  listen_ipv6=YES
  anonymous_enable=NO
  local_enable=YES
  write_enable=YES
  local_umask=022
  dirmessage_enable=YES
  use_localtime=YES
  xferlog_enable=YES
  connect_from_port_20=YES
  xferlog_file=/var/log/vsftpd.log
  xferlog_std_format=YES
  ftpd_banner=Welcome to Snail FTP service.
  chroot_list_enable=YES
  chroot_list_file=/etc/vsftpd.chroot_list
  secure_chroot_dir=/var/run/vsftpd/empty
  pam_service_name=ftp
  rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
  rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
  ssl_enable=NO
  utf8_filesystem=YES
  userlist_enable=YES
  userlist_deny=NO
  userlist_file=/etc/vsftpd.user_list
  allow_writeable_chroot=YES
  ```

- **创建FTP目录和登陆用户**

  ```shell
  #先创建ftp目录
  $ sudo mkdir /home/ftp
  #添加用户
  $ sudo useradd -d /home/ftp -s /bin/bash ftpuser
  #设置用户密码
  $ sudo passwd ftpuser
  #设置ftp目录用户权限
  $ sudo chown ftpuser:ftpuser /home/ftp
  #在etc目录下新建文件vsftpd.user_list，用于存放允许访问ftp的用户
  $ sudo touch /etc/vsftpd.user_list
  #编辑vsftpd.user_list，在其中添加ftpuser
  $ sudo vim /etc/vsftpd.user_list
  #在etc目录下新建文件vsftpd.chroot_list，设置可列出、切换目录的用户
  $ sudo touch /etc/vsftpd.chroot_list
  #编辑chroot_list，在其中添加ftpuser
  $ sudo vim /etc/vsftpd.chroot_list
  ```

- **重启vsftpd服务**

  ```shell
  $ sudo service vsftpd restart
  ```

配置完成后即可利用FileZilla在本地通过FTP连接服务器

![](D:\Document\BaiduYun\BaiduYun\capture_20190709204338626.bmp)

#### 在服务器中安装Anaconda

折腾Ubuntu服务器的初衷是学习Python，为了省心，直接在服务器上安装Anaconda。

> **Anaconda**是一个开源的包、环境管理器，含了conda、Python等180多个科学包及其依赖项，可以用于在同一个机器上安装不同版本的软件包及其依赖，并能够在不同的环境之间切换。

安装时最新的Anaconda安装包约654 MB，可以下载至本地后通过前面搭建的FTP服务传输至服务器（该方法似乎有点自找麻烦），也可以直接在服务器中运行下面语句下载。

```shell
$ wget https://repo.anaconda.com/archive/Anaconda3-2019.03-Linux-x86_64.sh
```

下载完成后，在文件所在目录运行如下语句，按提示安装即可。

```shell
$ bash Anaconda3-2018.12-Linux-x86_64.sh
```

安装完成后在终端中输入python，可以看到运行的python版本信息已经从此前系统默认的python2.7变为了python3.7，同时出现了 `Anaconda, Inc. on linux`信息。

![](D:\Document\BaiduYun\BaiduYun\capture_20190709210335081.bmp)

PS：如果此时运行python还是运行默认的python2.7，则在终端中运行如下语句即可解决。

```shell
$ source ~/.bashrc
```

#### 配置jupyter notebook，实现远程登录

> **Jupyter Notebook**是一个交互式笔记本，支持运行 40 多种编程语言，其本质是一个 Web 应用程序，便于创建和共享文学化程序文档，支持实时代码，数学方程，可视化和 markdown，用途包括数据清理和转换，数值模拟，统计建模，机器学习等等。

- **生成jupyter notebook配置文件**

  ```shell
  $ jupyter notebook --generate-config
  ```
- **设置登陆密码**

  ```shell
  $ jupyter notebook password
  ```

- **修改配置文件**

  ```shell
  $ vim ~/.jupyter/jupyter_notebook_config.py
  ```

  具体配置信息如下

  ```python
  c.NotebookApp.allow_remote_access = True
  c.NotebookApp.allow_root = True
  c.NotebookApp.ip = '*'
  c.NotebookApp.notebook_dir = '/home/snail/data/jupyter'
  c.NotebookApp.open_browser = False
  c.NotebookApp.port = 8888
  ```

- **配置百度云安全组**

  在百度云中创建安全组，将jupyter的端口8888加入到入站规则中，然后在实例中关联所创建的安全组即可。

- **在服务器后台运行jupyter notebook**

  如果直接在服务器终端中输入jupyter notebook命令，则终端关闭后服务即停止运行，因此如果想实现对服务器的随时访问，需让jupyter notebook后台运行。

  ```shell
  $ nohup jupyter notebook &
  ```

  其中，`nohup` 指的是不挂起，而`&`是后台运行

  PS：如果后续想关闭后台运行，则执行`ps -ef | grep jupyter`命令查询进程PID，然后执行`kill -9 查询到的进程PID`命令杀掉进程即可。

在浏览器中输入公网IP即可访问服务器jupyter notebook

![](D:\Document\BaiduYun\BaiduYun\capture_20190709214158334.bmp)