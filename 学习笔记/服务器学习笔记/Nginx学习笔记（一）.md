# Nginx学习笔记（一）

## 主要内容

Windows10下Nginx的基本配置与使用
[toc]
安装前的准备

需要提前安装 pcre，zlib 和 OpenSSL

pcre：是一个兼容正则表达式库，Nginx 的 Rewrite模块个 http 核心模块都用到了 pcre 正则表达式语法。

    安装命令：yum install -y pcre pcre-devel
    检测成功：rpm -qa pcre pcre-devel

zlib：提供了压缩算法，Nginx 各模块需要使用 gzip 压缩

```
安装命令：yum install -y zlib zlib-devel
检测成功：rpm -qa zlib zlib-devel
```

OpenSSL：是一个开源代码软件库报，使用该报可以进行安全通信，避免被窃听

```
安装命令：yum install -y openssl openssl-devel
检测成功：rpm -qa openssl openssl-devel
```

也可以直接通过一条命令安装：

```
yum install -y gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

## 下载安装

### 下载

[官网传送门](https://nginx.org/en/download.html)

![image-20210728152809229](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210813193701.png)
stable 为稳定版本

Nginx 免安装，解压便可以直接使用

### 启动方式

* 直接双击该目录下的"nginx.exe"，即可启动nginx服务器；
* 命令行进入该文件夹，执行start nginx命令，也会直接启动nginx服务器。

访问localhost，启动成功
![image-20210728152836506](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210813193655.png)

## Nginx Windows基本命令

```
启动服务：start nginx
退出服务：nginx -s quit
强制关闭服务：nginx -s stop
重载服务：nginx -s reload　　（重载服务配置文件，类似于重启，服务不会中止）
验证配置文件：nginx -t
使用配置文件：nginx -c "配置文件路径"
使用帮助：nginx -h
```
当命令行出现找不到Nginx命令时，可以试着在命令前加上.\ 具体原因在截图中
![image-20210728152913968](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210803095736.png)

## 平滑升级 Nginx

(1) 先进入到 sbin 目录下对 nginx 进行备份，后面如果搞砸了还可以恢复

    重命名 nginx 名字随意
    cd /usr/local/nginx/sbin
    mv ngin nginold

(2) 将新版 Nginx 安装目录（已经通过 ./configure 配置过且通过 make 编译过， 会生成 objs 目录）下 objs 下的 nginx，拷贝到原来 sbin 目录下

(3) 发送 kill USR2 pid 进行升级，会重新启动一个 master 和 worker 进程，这两个进程使用的就是新拷贝的 nginx 。同时会在 logs 目录下生成一个 nginx.pid.oldbin，记录了旧版本的 pd 号，用于后面的结束旧进程操作

(4) 最后在通过 kill -QUIT 旧进程pid 删除旧版本进程，此时在运行的就只剩下新版进程了。这就成功实现了平滑将 nginx 升级，而不用停止服务器服务器

最后可以通过 ./nginx -v 查看是否更新成功
这部分还会有简化版的操作，第 (1) (2) 步都一样
然后可以直接使用 make upgrade 代替第 (3) (4) 步，效果一样（其实就是帮我们做了这两步 ）
注意：这一步的执行需要在根目录下（nginx-1.xxx）

## 配置文件（nginx.conf）

配置文件位置：(usr/local/) nginx/conf/nginx.conf

> 总共分为三大块：全局快，events 块，http 块
> http 块中可以配置多个 server 块
> 每个 server 块又可以配置多个 location 块

**注意了：配置文件的所有语句后面都有分号 ; 每次修改完配置文件都要重新加载**

### 全局块

- `user xxx` 可以设置用户，默认不为 nobody
- `master_process off` 关闭工作线程（默认为 on，即开启工作线程），注意了这个需要重启 nginx 服务器，否则不会生效
- `worker_processes` 可以设置工作线程数，理论上越多处理并发的能力越强，但这也是受限于服务器的，一般最大数量设置为服务器内核数
- `daemon off` 关闭守护进程，默认为 on，设置为关闭之后。nginx 将随启动它的终端的关闭而关闭，并且启动它的终端会被阻塞。而设置为 on 的话，nginx 则会一直存在，且启动它的终端可以做其他的事情而不被阻塞
- `pid xxx` 指定 master 进程 id 号存放的文件路径（nginx.pid），默认为 /usr/local/nginx/logs/nginx.pid
- `error_log file [日志级别]` 可以指定 nginx 错误日志的存放路径，还可以指定日志输出的级别，默认值为 logs/error.log error。**（也可以在 http server location 块中出现）**日志级别有：debug | info | notice | warn | error | crit | alert | emerg，从左往右级别依次升高，输出内容变少
- `include xxx` 可以引入其他的配置文件，使配置更为灵活 **（可以在任何地方使用）**

### events块

主要用于设置 nginx 服务器和用户的网络连接，对 nginx 服务器的性能影响较大

- accept_mutex off 关闭网络连接的序列化，默认为 on。on 则表示网络连接是序列化的，当有连接请求发到服务器时，这些请求会按顺序被 worker 进程接收。而设置为 off 则表示有请求过来时，全部 worker 进程全部被激活，共同接收这些请求。显然，在连接量小的时候，设置为 off 比较合理，而连接量大时，应该设置为 off
- `multi accept on` 允许一个工作进程同时接收多个连接，默认为 off 即一个工作进程只能接收一个连接
- `worker_connections <num>` 设置单个工作进程的最大连接数（包括了可能的所有连接）
- `use epoll` 使用 epoll 这个事件驱动来处理网络信息（linux 2.6 以上可以用，效率较高）

### http块

- access_log path[format[buffer=size]] 设置用户访问日志的相关属性，包括路径，输出格式和缓存的大小，默认值为 logs/access.log combined，combined 为下面所指定的那个 （可以在 http server location 使用）
- `log_format name[escape=default|json|none] string ...]` 指定日志的输出格式，默认值为 combined “…”
- `sendfile on` 设置 nginx 服务器使用 sendfile() 传输文件，可以大大提高 ngins 处理静态资源的能力，默认为 off ，建议打开
- `keepalive_timeout <num>` 设置长连接失效的秒数，默认为 75 秒