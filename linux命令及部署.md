### 安装linux

#### VMware 虚拟机安装

> 如果是win10关闭Hiper-V虚拟机
>
> ​	win+i --> 程序  -->  启用或关闭windows功能  --> 找到 Hyper-V 取消勾选
>
> 安装一直下一步即可

#### 系统安装

> Linux 有收费的 红帽： Red Hat
> 免费的常见的有： Ubuntu，CentOS，Debian 。
> 他们区别大概是：
> Ubuntu 界面好看
> CentOS 文档丰富
> Debian 稳定性强

SentOS 镜像地址

``` text
http://mirrors.cn99.com/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://ftp.sjtu.edu.cn/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://mirrors.nwsuaf.edu.cn/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://mirrors.cqu.edu.cn/CentOS/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://mirrors.163.com/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://mirrors.aliyun.com/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://mirrors.neusoft.edu.cn/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://mirrors.shu.edu.cn/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://mirrors.tuna.tsinghua.edu.cn/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://mirror.lzu.edu.cn/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://mirrors.nju.edu.cn/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://mirror.jdcloud.com/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://mirrors.huaweicloud.com/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
http://mirrors.zju.edu.cn/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso
```

创建虚拟机

> 选择典型 -->  稍后安装操作系统 -->  选择CentOS 7 64位 -->  选择存储位置 -->  磁盘容量默认即可 -->  自定义硬件中(内存2G、一个处理器，虚拟化 Intel VT-x/EPT 或者 AMD-V/RVI(V)选上、指定镜像位置 )

运行虚拟机开始安装

> 语言版本选择中文 -->  安装信息摘要(安装位置，进去后什么都不动点击完成、网络和主机名，打开以太网) -->  点击开始安装 -->  设置root的密码

链接ssh，使用ip address命令 找到虚拟ip，使用ssh工具链接即可

安装可视化，安装系统时默认最小安装，需要可视化可以执行下面命令

``` shell
$sudo  yum groupinstall "GNOME Desktop" "Graphical Administration Tools"
（出现下面错误）
Transaction check error:
  file /usr/lib/systemd/system/blk-availability.service from install of device-mapper-7:1.02.107-5.el7_2.2.x86_64 conflicts with file from package lvm2-7:2.02.105-14.el7.x86_64
  file /usr/sbin/blkdeactivate from install of device-mapper-7:1.02.107-5.el7_2.2.x86_64 conflicts with file from package lvm2-7:2.02.105-14.el7.x86_64
  file /usr/share/man/man8/blkdeactivate.8.gz from install of device-mapper-7:1.02.107-5.el7_2.2.x86_64 conflicts with file from package lvm2-7:2.02.105-14.el7.x86_64
 
Error Summary

（需要先执行以下两条安装命令）
yum install -y libdevmapper*
yum install -y docker
（安装完成后重新执行上面的安装 Gnome包命令）

systemctl get-default #获取当前系统运行形式，会显示multi-user.target（命令行终端），或者：graphical.target（可视化）
systemctl set-default graphical.target #设置默认启动为图形界面，reboot后界面会自动是图形窗口了。
systemctl set-default multi-user.target #换回命令界面启动
reboot #重启
```

常用插件的安装

``` shell
yum install iproute ftp bind-utils net-tools wget -y
#iproute 用来执行 ip address 查看本机地址
#ftp 用来测试ftp 服务器 ftp 127.0.0.1
#bind_utils 用来运行 nslookup www.baidu.com
#net-tools 用来执行 netstate -anp|grep 端口号
#wget 获取网络资源 wget 下载地址
```

设置静态ip

确保当前模式是 NAT 模式

![](/imgs/9152.png)

菜单 --> 编辑 --> 虚拟网络编辑器 --> 更改设置

![](/imgs/9153.png)

1. 算中 VMnet8
2. 取消勾选 使用本地 DHCP 服务将 IP 地址分配给虚拟机
3. 设置子网 ip: 192.168.2.0
4. 点击 NAT 设置

![](/imgs/9154.png)

修改 IP 地址为 :192.168.2.1

![](/imgs/9155.png)

点击确定

win10 修改VMnet8的ip地址为：192.168.2.3

![](/imgs/ip-update.gif)

在SentOS中设置

``` shell
cd /etc/sysconfig/network-scripts/
ls
```

![](/imgs/9161.png)

``` shell
使用如下命令编辑此文件，linux版本不同后缀的33会不同，选择前面正确的文件即可
vi ifcfg-ens33
修改一个地方：
BOOTPROTO修改为下图所示的 static
然后在下面追加4行
#DNS地址
DNS1=114.114.114.114
#本机ip地址   
IPADDR=192.168.2.2      
#子网掩码
NETMASK=255.255.255.0
#网关地址，在NAT 设置里设置的地址
GATEWAY=192.168.2.1
#保存编辑
esc键 :wq
#重启网络服务
service network restart
#查看ip是否更改
ip address
#检测网络是否通畅
ping www.baidu.com
#静态ip修改完毕，可以使用ssh链接
```

![](/imgs/9162.png)

### Linux 命令记录及使用

> linux 系统 + java + node + npm + yarn （环境中可以使用nohup 命令）

#### 项目部署前后端项目

``` java
which nohup //查看环境变量中符合条件的nohup
//记录日志文件，在前端项目目录下创建运行
touch my.log
chmod u+w my.log
nohup npm run dev > my.log 2>my.log &
//查看进程,运行则成功
ps -ef|grep node
//不记录日志的运行方式
nohup npm run dev >/dev/null 2>&1 &
//运行nohup后台运行后一定使用exit登出控制台，否则会关闭
```

#### 服务器部署node

> 英文网址：https://nodejs.org/en/download/
>
>  中文网址：http://nodejs.cn/download/

``` java
uname -a //查看系统位数（x86_64表示64位系统， i686 i386表示32位系统）
tar -xvf   node-v6.10.0-linux-x64.tar.xz   //解压文件
mv node-v6.10.0-linux-x64  nodejs //修改文件夹名称
ln -s /app/software/nodejs/bin/npm /usr/local/bin/ //建立软连接,/app/software 是node的文件目录
ln -s /app/software/nodejs/bin/node /usr/local/bin/ //建立成功之后再/usr/local/bin下查看，如果为红色说明没有建立正确，删除并重新建立
node -v //检查node是否安装
npm install -g yarn //安装yarn 比npm速度更快，安全可靠
yarn --version //查看版本或 yarn -v
//替换yarn为淘宝源
yarn config set registry https://registry.npm.taobao.org -g 
yarn config set sass_binary_site http://cdn.npm.taobao.org/dist/node-sass -g
```

##### 常用npm与yarn对比

| 作用           | npm                                | Yarn                  |
| -------------- | ---------------------------------- | --------------------- |
| 安装           | npm install(i)                     | yarn                  |
| 卸载           | npm uninstall(un)                  | yarn remove           |
| 全局安装       | npm install xxx –-global(-g)       | yarn global add xxx   |
| 安装包         | npm install xxx –save(-S)          | yarn add xxx          |
| 开发模式安装包 | npm install xxx –save-dev(-D)      | yarn add xxx –dev(-D) |
| 更新           | npm update –save                   | yarn upgrade          |
| 全局更新       | npm update –global                 | yarn global upgrade   |
| 卸载           | npm uninstall [–save/–save-dev]    | yarn remove xx        |
| 清除缓存       | npm cache clean                    | yarn cache clean      |
| 重装           | rm -rf node_modules && npm install | yarn upgrade          |

