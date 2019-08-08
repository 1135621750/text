#### 1.下载安装包

> https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/  清华大学镜像站查找最新的版本

``` java
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-12.1.4-ce.0.el7.x86_64.rpm
```

#### 2.安装

``` java
yum -y install gitlab-ce-12.1.4-ce.0.el7.x86_64.rpm
```

#### 3.在 Centos 6 和 7 系统上，下面的命令将在系统防火墙里面开放 HTTP 和 SSH 端口

``` java
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo systemctl reload firewalld
```

#### 4.邮件服务，可选

``` java
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

#### 5.修改访问地址 external_url 'http://172.16.4.74’

``` java
vi /etc/gitlab/gitlab.rb
```
重新加载配置
``` java
gitlab-ctl reconfigure
```

#### 6.访问
如果本机可以访问其余电脑不能访问，则开启对应防火墙端口

如果出现502

``` java
chmod -R 755 /var/log/gitlab
```

#### 7.修改密码
第一次访问页面时会修改root密码，修改后则可以登录


#### 8.汉化



#### 常用命令及目录

``` java
gitlab-ctl start                                 # 启动 gitlab
gitlab-ctl stop                                  # 停止 gitlab
gitlab-ctl restart                               # 重启 gitlab
gitlab-ctl status                                # 查看服务状态
vi /etc/gitlab/gitlab.rb                         # 修改配置文件
gitlab-ctl reconfigure                           # 重新编译 gitlab 配置
gitlab-rake gitlab:check SANITIZE=true --trace   # 检查 gitlab
gitlab-ctl tail                                  # 查看日志
gitlab-ctl tail nginx/gitlab_access.log
```

``` java
/var/log/gitlab/                                 # 日志地址 
/var/opt/gitlab/                                 # 服务地址 
```

``` java
cat /opt/gitlab/embedded/service/gitlab-rails/VERSION #查看版本
```
