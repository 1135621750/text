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

