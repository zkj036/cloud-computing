

## 实验一

![](./image/1.png)

![](./image/2.png)

![](./image/3.png)

![](./image/4.png)

![](./image/5.png)

![](./image/6.png)

![](./image/7.png)

![](./image/8.png)

![](./image/9.png)

![](./image/10.png)

实验二

## 1.安装Apache Web服务器

使用yum工具安装：

sudo yum install httpd

sudo命令获得了root用户的执行权限，因此需要验证用户口令。
安装完成之后，启动Apache Web服务器：

sudo systemctl start httpd.service

![](./image/11.png)

测试Apache服务器是否成功运行，找到腾讯云实例的公有IP地址(your_cvm_ip)，在你本地主机的浏览器上输入：

```
http://your_cvm_ip/
```

若运行正常，将出现如下界面：

![](./image/12.png)

## 2.安装MySQL

CentOS 7.2的yum源中并末包含MySQL，需要其他方式手动安装。因此，我们采用MySQL数据库的开源分支MariaDB作为替代。
安装MariaDB：

```
sudo yum install mariadb-server mariadb
```

![](./image/13.png)

安装好之后，启动mariadb：

```
sudo systemctl start mariadb
```

![](./image/14.png)

随后，运行简单的安全脚本以移除潜在的安全风险，启动交互脚本：

sudo mysql_secure_installation
1
设置相应的root访问密码以及相关的设置(都选择Y)。
最后设置开机启动MariaDB：

sudo systemctl enable mariadb.service

![](./image/15.png)

![](./image/16.png)

3.安装PHP
PHP是一种网页开发语言，能够运行脚本，连接MySQL数据库，并显示动态网页内容。
默认的PHP版本太低（PHP 5.4.16），无法支持最新的WordPress（笔者写作时为5.2.2），因此需要手动安装PHP较新的版本(PHP 7.2)。
PHP 7.x包在许多仓库中都包含，这里我们使用Remi仓库，而Remi仓库依赖于EPEL仓库，因此首先启用这两个仓库

sudo yum install epel-release yum-utils
sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm

![](./image/17.png)

![](./image/18.png)

![](./image/19.png)

![](./image/20.png)

![](./image/21.png)

接着启用PHP 7.2 Remi仓库：

sudo yum-config-manager --enable remi-php72

![](./image/22.png)

安装PHP以及php-mysql

sudo yum install php php-mysql

![](./image/23.png)

![](./image/24.png)

查看安装的php版本：

php -v

![](./image/25.png)

安装之后，重启Apache服务器以支持PHP：

sudo systemctl restart httpd.service

![](./image/26.png)

### 安装PHP模块

为了更好的运行PHP，需要启动PHP附加模块，使用如下命令可以查看可用模块：

```
yum search php-
```

![](./image/27.png)

这里先行安装php-fpm(PHP FastCGI Process Manager)和php-gd(A module for PHP applications for using the gd graphics library)，WordPress使用php-gd进行图片的缩放。

sudo yum install php-fpm php-gd

![](./image/28.png)

![](./image/29.png)

重启Apache服务：

sudo service httpd restart

![](./image/30.png)

至此，LAMP环境已经安装成功，接下来测试PHP。


4.测试PHP
这里我们利用一个简单的信息显示页面（info.php）测试PHP。创建info.php并将其置于Web服务的根目录（/var/www/html/）：

sudo vim /var/www/html/info.php

![](./image/31.png)

该命令使用vim在/var/www/html/处创建一个空白文件info.php，我们添加如下内容：

![](./image/32.png)

<?php phpinfo(); ?>
完成之后，使用刚才获取的cvm的IP地址，在你的本地主机的浏览器中输入:

http://your_cvm_ip/info.php

![](./image/33.png)

## 5.安装WordPress以及完成相关配置

### (1)为WordPress创建一个MySQL数据库

首先以root用户登录MySQL数据库：

```
mysql -u root -p
```

![](./image/34.png)

首先为WordPress创建一个新的数据库：

CREATE DATABASE wordpress;

![](./image/35.png)

注意：MySQL的语句都以分号结尾。
接着为WordPress创建一个独立的MySQL用户：

CREATE USER wordpressuser@localhost IDENTIFIED BY 'password';

![](./image/36.png)

“wordpressuser”和“password”使用你自定义的用户名和密码。授权给wordpressuser用户访问数据库的权限：

![](./image/37.png)

GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost IDENTIFIED BY 'password';
随后刷新MySQL的权限：

FLUSH PRIVILEGES;

![](./image/38.png)

最后，退出MySQL的命

令行模式：

exit

![](./image/39.png)

(2)安装WordPress
下载WordPress至当前用户的主目录：

cd ~
wget http://wordpress.org/latest.tar.gz

wget命令从WordPress官方网站下载最新的WordPress集成压缩包，解压该文件：

tar xzvf latest.tar.gz

解压之后在主目录下产生一个wordpress文件夹。我们将该文件夹下的内容同步到Apache服务器的根目录下，使得wordpress的内容能够被访问。这里使用rsync命令：

sudo rsync -avP ~/wordpress/ /var/www/html/

![](./image/40.png)

接着在Apache服务器目录下为wordpress创建一个文件夹来保存上传的文件：

mkdir /var/www/html/wp-content/uploads

![](./image/41.png)

对Apache服务器的目录以及wordpress相关文件夹设置访问权限：

sudo chown -R apache:apache /var/www/html/*

![](./image/42.png)

这样Apache Web服务器能够创建、更改WordPress相关文件，同时我们也能够上传文件。

(3)配置WordPress
大多数的WordPress配置可以通过其Web页面完成，但首先通过命令行连接WordPress和MySQL。
定位到wordpress所在文件夹：

cd /var/www/html

WordPress的配置依赖于wp-config.php文件，当前该文件夹下并没有该文件，我们通过拷贝wp-config-sample.php文件来生成：

cp wp-config-sample.php wp-config.php

![](./image/43.png)

然后，通过nano超简单文本编辑器来修改配置，主要是MySQL相关配置：

nano wp-config.php

![](./image/44.png)

### (4)通过Web界面进一步配置WordPress

经过上述的安装和配置，WordPress运行的相关组件已经就绪，接下来通过WordPress提供的Web页面进一步配置。输入你的IP地址或者域名：

```
http://server_domain_name_or_IP
```

出现如下界面：

![](./image/45.png)

设置网站的标题，用户名和密码以及电子邮件等，点击**Install WordPress**，弹出确认页面：

![](./image/46.png)

点击**Log In**，弹出登录界面：

![](./image/48.png)

输入用户名和密码之后，进入WordPress的控制面板：

![](./image/47.png)





实验三

使用如下命令查看操作系统内核信息：

uname -r

![](./image/49.png)

顺带看一下Linux的版本号：

```
cat /etc/redhat-release
```

![](./image/50.png)

**安装Docker**

CentOS 7的应用程序库可能不是最新的，因此首先更新应用程序数据库：

sudo yum check-update

![](./image/51.png)

接下来添加Docker的官方仓库，下载最新的Docker并安装：

```
curl -fsSL https://get.docker.com/ | sh
```

![](./image/52.png)

安装完成之后启动Docker守护进程，即Docker服务：

sudo systemctl start docker

验证Docker是否成功启动：

sudo systemctl status docker

得到类似如下图的输出：

![](./image/53.png)

最后，确保Docker当服务器启动时自启动：

```
sudo systemctl enable docker

```

![](./image/54.png)

此外，还可以查看一下Docker的版本信息：

```
docker version
```

![](./image/55.png)

# Docker基本操作

查看docker所有的命令，键入：

docker

![](./image/56.png)

查看当前系统docker的相关信息：

docker info

![](./image/57.png)

## 加载Docker镜像

Docker镜像是容器运行的基础，默认情况下，将从Docker Hub拉取镜像。首先使用search命令查询Docker Hub中的可用镜像，这里以查询可用的CentOS镜像为例：

docker search centos

![](./image/58.png)

接下来拉取官方版本(OFFICIAL)的镜像：

docker pull centos：7

![](./image/59.png)

一旦镜像下载完成，可以基于该镜像运行容器，使用run命令：

docker run centos

查看一下当前系统中存在的镜像：

docker images

![](./image/60.png)

 

## 运行Docker容器

以上述的CentOS镜像为例运行其容器，使用-it参数进入交互shell模式：

docker run -it centos

进行container内部shell，如下图所示：

## ![](./image/61.png)

## 创建新的镜像

在前序操作的基础上，本小节将创建新的镜像，即提交更改到新的镜像。首先从容器的交互shell退出并保存状态，使用exit命令

exit

![](./image/62.png)

我们首先使用如下命令查看本地中的容器：

docker ps -a

![](./image/63.png)

现在使用commit命令来提交更改到新的镜像中，即创建新的镜像。命令格式

docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

这种提交类似于git协议的提交，同样这里提交的镜像只保存在本地。后续可以提交到远程镜像仓库，比如Docker Hub。
 再次使用镜像查看命令：

docker images

![](./image/64.png)

运行Docker容器（为了方便检测后续wordpress搭建是否成功，需设置端口映射（-p），将容器端口80 映射到主机端口8888，Apache和MySQL需要 systemctl 管理服务启动，需要加上参数 –privileged 来增加权，并且不能使用默认的bash，换成 init，否则会提示 Failed to get D-Bus connection: Operation not permitted ，-name 容器名 ，命令如下 ）

docker run -d -it --privileged --name wordpress -p 8888:80 -d centos:7 /usr/sbin/init

查看已启动的容器

docker ps

![](./image/65.png)

进入容器前台：

![](./image/66.png)

开始安装WordPress：

安装Apache Web服务器

![](./image/67.png)

安装成功：

![](./image/68.png)

*安装完成后，启动**Apache Web**服务器：*

*设置开机自启：*

![](./image/69.png)

访问公网ip(加上端口:8888)：

### ![](./image/70.png)安装MySQL

*安装MariaDB*

```
yum install mariadb-server mariadb

安装成功：

```

![](./image/71.png)

![](./image/72.png)

启动MariaDB*

```
systemctl start mariadb

```

![](./image/73.png)

设置MySQL的root密码*

```
mysql_secure_installation

```

初始密码为空，提示输入正确密码，直接回车，再设置密码，其他选择Y

![](./image/75.png)

![](./image/76.png)

![](./image/77.png)

*设置开机自启MariaDB*

```
systemctl enable mariadb.service
```

![](./image/78.png)

### 安装PHP

```
yum install epel-release yum-utils


yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm


```

![](./image/79.png)

![](./image/80.png)

![](./image/81.png)

![](./image/82.png)

因为WordPress需要php5.6以上版本的支持，我们更新到7.2版本仓库*

```
yum-config-manager --enable remi-php72


```

![](./image/83.png)

![](./image/84.png)



安装PHP以及php-mysql*

```
yum install php php-mysql

安装成功：

```

![](./image/85.png)

![](./image/86.png)

查看安装的php版本*

```
php -v

```

![](./image/87.png)

重启Apache服务器以支持PHP*

```
systemctl restart httpd.service

```

![](./image/88.png)

为了更好的运行PHP，需要启动PHP附加模块*

```
yum install php-fpm php-gd

安装成功:

```

![](./image/89.png)

![](./image/90.png)

重启Apache服务*

```
systemctl restart httpd.service
```

![](./image/91.png)

### 安装WordPress以及完成相关配置

*登录数据库*

```
mysql -u root -p
```

![](./image/92.png)

*为WordPress创建一个新的数据库*

```
CREATE DATABASE 数据库名 ;
```

![](./image/93.png)

*进入刚创建的数据库*

```
use 数据库名 ;
```

为WordPress创建一个独立的MySQL用户并授权给数据库访问权限*

```
CREATE USER 用户名@localhost IDENTIFIED BY '密码';

GRANT ALL PRIVILEGES ON WordPress.* TO zkj@localhost IDENTIFIED BY '123456';

```

![](./image/94.png)

![](./image/95.png)

刷新MySQL的权限*

```
FLUSH PRIVILEGES;
```

![](./image/96.png)

*安装WordPress*

```
git clone https://gitee.com/helang_z/wordpress.git
```

![](./image/97.png)

sudo rsync -avP ~/wordpress/ /var/www/html/

![](./image/98.png)

mkdir /var/www/html/wp-content/uploads

sudo chown -R apache:apache /var/www/html/*

![](./image/99.png)

 

配置WordPress

大多数的WordPress配置可以通过其Web页面完成，但首先通过命令行连接WordPress和MySQL。

定位到wordpress所在文件夹：

cd /var/www/html

![](./image/100.png)

WordPress的配置依赖于wp-config.php文件，当前该文件夹下并没有该文件，我们通过拷贝wp-config-sample.php文件来生成：

cp wp-config-sample.php wp-config.php

![](./image/101.png)

然后，通过vi wp-config.php，主要是MySQL相关配置：

vi wp-config.php

![](./image/102.png)

输入自己的公网ip加端口：http://106.54.30.215:8888/

出现以下画面：

![](./image/103.png)

![](./image/104.png)

成功进入WordPress：

![](./image/105.png)

进一步细化到推送至Docker Hub的镜像，并查看镜像：

docker tag 67fa590cfc1c zkj035/centos:7

docker images

![](./image/106.png)

首先要到[Docker Hub](https://hub.docker.com/)上进行注册，然后这里我们使用shell登录：

docker login -u docker-hub-username

输入密码。用户名和密码都正确，随后会显示登录成功。

![](./image/107.png)

使用如下命令推送新创建的镜像：

![](./image/108.png)

进入docker hub查看镜像

![](./image/109.png)

Dockerfile实验：

（参考何浪同学的dockerfile思路）

编写dockerfile文件：

![](./image/110.png)

编写install.sh文件：

![](./image/111.png)

编写setup.sql文件：

![](./image/112.png)

编写start.sh文件：

![](./image/113.png)

执行dockerfile文件(一定要注意结尾是空格加一个  .  ）

docker build -t dockerfile .

![](./image/114.png)

启动容器:docker run -dit dockerfile

![](./image/115.png)

打开浏览器验证wordpress:

106.54.30.215:9998

![](./image/116.png)

填写个人信息后成功进入wordpress界面：

![](./image/118.png)



实验六：ceph

在虚拟机安装centos（最小化安装）

进入界面，里头无法使用复制粘贴所以需要用shell连接方便操作：

输入ip addr  查看ip

![](./image/120.png)

获取ip后连接shell

![](./image/119.png)

打开shell

在所有节点上创建一个名为“ **cephuser** ” 的新用户。

```
useradd -d /home/cephuser -m cephuser
passwd cephuser
```

![](./image/121.png)

创建新用户后，我们需要为“ cephuser”配置sudo。他必须能够以root用户身份运行命令，并且无需密码即可获得root用户特权。

运行以下命令为用户创建一个sudoers文件，并使用sed编辑/ etc / sudoers文件。

```
echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
chmod 0440 /etc/sudoers.d/cephuser
sed -i s'/Defaults requiretty/#Defaults requiretty'/g /etc/sudoers
```

![](./image/122.png)

安装NTP以同步所有节点上的日期和时间。运行ntpdate命令通过NTP协议设置日期和时间，我们将使用us pool NTP服务器。然后启动并启用NTP服务器在引导时运行。

```
yum install -y ntp ntpdate ntp-doc
ntpdate 0.us.pool.ntp.org
hwclock --systohc
systemctl enable ntpd.service
systemctl start ntpd.service
```

![](./image/123.png)

![](./image/124.png)

![](./image/125.png)

### 安装Open-vm-tools

```
yum install -y open-vm-tools
```

![](./image/126.png)

![](./image/127.png)

### 禁用SELinux

通过使用sed流编辑器编辑SELinux配置文件，在所有节点上禁用SELinux。

```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

![](./image/128.png)

### 配置主机文件

使用vim编辑器在所有节点上编辑/ etc / hosts文件，并添加带有所有集群节点的IP地址和主机名的行。

```
vim /etc/hosts
```

![](./image/131.png)

保存文件并退出vim。

现在，您可以尝试使用主机名在服务器之间ping通，以测试网络连接。例：

```
ping -c 4 mon1
```

![](./image/132.png)

## 第2步-配置SSH服务器

在此步骤中，我将配置**ceph-admin节点**。admin节点用于配置监视节点和osd节点。登录到**ceph** -admin节点并成为“ **cephuser** ”。

```
ssh root@ceph-admin
然后切换用户
su - cephuser
```



admin节点用于安装和配置所有群集节点，因此ceph-admin节点上的用户必须具有无需密码即可连接到所有节点的特权。我们必须在“ ceph-admin”节点上为“ cephuser”配置无密码的SSH访问。

为“ **cephuser** ” 生成ssh密钥。

```
ssh-keygen
```

将密码短语留空/空白。

![](./image/133.png)

接下来，为ssh配置创建配置文件。

```
vim ~/.ssh/config
```

粘贴以下配置：

```
Host ceph-admin        
Hostname ceph-admin        
User cephuser Host mon1        
Hostname mon1        
User cephuser Host osd1        
Hostname osd1        
User cephuser Host osd2        
Hostname osd2        
User cephuser
```

![](./image/134.png)

更改配置文件的权限。

```
chmod 644 ~/.ssh/config
```

![](./image/136.png)

现在，使用ssh-copy-id命令将SSH密钥添加到所有节点。

```
ssh-keyscan osd1 osd2 mon1 >> ~/.ssh/known_hosts
ssh-copy-id osd1
ssh-copy-id osd2
ssh-copy-id mon1
```

![](./image/137.png)

![](./image/138.png)

完成后，请尝试从ceph-admin节点访问osd1服务器。

```
ssh osd1
ssh osd2
ssh mon1
```

![](./image/145.png)

由于本地操作，就没有做防火墙相关操作

## 构建Ceph集群

在这一步中，我们将在ceph-admin节点的所有节点上安装Ceph。

登录到ceph-admin节点。

```
ssh root@ceph-adminsu - cephuser
```

 

### 在ceph-admin节点上安装ceph-deploy

添加Ceph存储库，并使用yum命令安装Ceph部署工具' **ceph-deploy** '。

```
sudo rpm -Uhv http://download.ceph.com/rpm-jewel/el7/noarch/ceph-release-1-1.el7.noarch.rpm
sudo yum update -y && sudo yum install ceph-deploy -y
```

![](./image/140.png)

在osd1 osd2 mon1上重复上面步骤

安装ceph-deploy工具后，为ceph集群配置创建一个新目录。



```
mkdir cluster
cd cluster/
```

![](./image/141.png)

接下来，使用“ **ceph** **-deploy** ”命令创建一个新的集群配置，将监视节点定义为“ **mon1** ”。

```
ceph-deploy new mon1
```



```
mkdir cluster
cd cluster/
```

![](./image/142.png)

该命令将在集群目录中生成Ceph集群配置文件'ceph.conf'。

用vim编辑ceph.conf文件。

```
vim ceph.conf
```

在[global]块下，在下面粘贴配置。

```
# Your network addresspublic network = 192.168.40.0/24（因人而异，具体看自己的ip）
osd pool default size = 2
```

![](./image/143.png)

现在，从ceph-admin节点在所有其他节点上安装Ceph。这可以通过单个命令完成。

```
ceph-deploy install ceph-admin mon1 osd1 osd2
```

![](./image/146.png)

![](./image/147.png)

该命令将在所有节点上自动安装Ceph：mon1，osd1-2和ceph-admin-安装将花费一些时间。

现在将ceph-mon部署在mon1节点上。

```
ceph-deploy mon create-initial
```

该命令将创建监视键，并使用“ ceph”命令检查并获取键。

```
ceph-deploy gatherkeys mon1
```

![](./image/149.png)

将OSDS添加到集群

```
ceph-deploy osd prepare osd1:/var/local/osd1 osd2:/var/local/osd2

ceph-deploy osd activate osd1:/var/local/osd1 osd2:/var/local/osd2
```

![](./image/150.png)

将管理密钥部署到所有关联的节点

将OSDS添加到集群

所有节点执行命令更改密钥文件的权限

```
ceph-deploy admin ceph-admin mon1 osd1 osd2

*sudo chmod 644 /etc/ceph/ceph.client.admin.keyring*

*ceph-deploy disk list osd1 osd2*
```

![](./image/151.png)

登录mon1管理节点，查看health状态

![](./image/152.png)

检查集群状态

![](./image/153.png)