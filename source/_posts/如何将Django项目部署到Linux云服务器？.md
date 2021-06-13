---
title: 如何将 Django 项目部署到Linux云服务器？
date: 2019-10-29 16:55:07
tags: 服务器
categories: 后端
---

**本文操作环境：** Debian 9，lnmp1.6，Django2.1.3，Python3.6.0

> 主要参考了文档 [查看](https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html)

### 1. 安装 Python3.6.0
由于Django2.1仅支持Python3.5以上的版本，所以需要安装Python3版本。如果直接安装Python3版本，使用时需要改变全局默认Python版本，还需要重新配置环境变量，所以推荐一个叫pyenv的Python多版本管理工具，它构建出一个Python虚拟环境，可以在多个Python版本间随意切换。

<!-- more -->

#### 1.1 安装pyenv（首先创建~/.pyenv目录）

官方给出了安装教程 https://github.com/pyenv/pyenv#installation

```bash
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
# 注意，文件名不一定叫.bash_profile，有可能是.bashrc
```

#### 1.2 安装完成之后下载Python3.6.0版本

使用命令：

```bash
pyenv install 3.6.0
```

这时pyenv会自动下载并将其放入版本集合里，如果想全局使用Python3.6.0，只需要执行：

```bash
pyenv global 3.6.0
```

### 2. 安装 Django 2.1.3

```bash
pip install Django==2.1.3
```

我根据自己的需要，还安装了一些其他库

```bash
pip install requests
pip install mysqlclient
```

### 3. 安装 uWSGI

由于Django并不推荐在其自带的开发环境下的WSGI模块运行，所以还需要安装uWSGI
> 参考：[如何用 uWSGI 托管 Django](https://docs.djangoproject.com/zh-hans/2.2/howto/deployment/wsgi/uwsgi/)

```bash
pip install uwsgi
```

完整的调用关系应该是这样的：

```bash
the web client <-> the web server <-> the socket <-> uwsgi <-> Django
```

这里uWSGI大致起到一个中间件的作用，它实现了Django的WSGI协议。客户端请求由Nginx处理分发到uwsgi，uwsgi再将请求交给Django处理，Django处理完成之后将结果原路返回，最后由Nginx返回数据给客户端。

### 4. 创建一个虚拟主机，修改网站配置文件
> 由于我的服务器上放了不止一个站点，所以需要创建虚拟主机，当然不创建也可以，无非就是访问的时候主域名 + 端口号

我使用的是**lnmp一键安装脚本**安装的Nginx，所以创建虚拟主机完全命令化：

```bash
lnmp vhost add
```
接下来会提示你配置一些关键信息

如果想要支持https，选择添加SSL安全认证时选用Let's Encript

```bash
Add SSL Certificate (y/n) y
1: Use your own SSL Certificate and Key
2: Use Let's Encrypt to create SSL Certificate and Key
```

uWSGI 文档提供了一个模板

```bash
# mysite_nginx.conf

# the upstream component nginx needs to connect to
upstream django {
    # server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    server 127.0.0.1:8001; # for a web port socket (we'll use this first)
    # 大概就是一次转发，与真正的 Django 应用通信
    # 二选一，unix套接字方式或端口
}

# configuration of the server
server {
    # the port your site will be served on
    listen      8000;
    # the domain name it will serve for
    server_name example.com; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # 下面两个主要就是 Django 项目下的一些静态资源，静态资源的话就可以直接交给nginx发送
    
    # Django media
    location /media  {
        alias /path/to/your/mysite/media;  
        # your Django project's media files - amend as required
    }

    location /static {
        alias /path/to/your/mysite/static; 
        # your Django project's static files - amend as required
        # 使用 manage.py 执行 collectstatic 命令可以整理出来
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /path/to/your/mysite/uwsgi_params; 
        # the uwsgi_params file you installed 
        # 这个文件大概在nginx的conf目录下，我把这个文件复制到了我项目下
    }
}
```
如果配置了https，那么这个文件下会看到有两个server模块，整体结构不用改变，参考上面文件 server 中的内容填到自己的配置文件中就好了。


### 5. 编写uWSGI配置文件

这里我给出一个示例

```bash
# book.ini
[uwsgi]

# Django-related settings
# the base directory (full path)
chdir           = /home/wwwroot/shuchong1
# Django's wsgi file
module          = shuchong1.wsgi
# the virtualenv (full path)
# home            = /path/to/virtualenv

# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 1
# the socket (use the full path to be safe

# 这里还是一样，可以使用端口号后者socket，但是注意要与Nginx配置文件保持一致

#http     = :8000
socket          = /home/wwwroot/shuchong1/shuchong1.sock

# ... with appropriate permissions - may be needed
chmod-socket    = 666 # 给 socket 添加权限
# clear environment on exit
vacuum          = true
enable-threads = true # 允许多线程
touch-reload = /home/wwwroot/shuchong1 
# 热重载配置，意思就是对应目录下的文件有更改，不用手动重启uWSGI
```
事实上，这个时候如果执行命令 uwsgi --ini book.ini 项目可以启动了

### 6. 最后一步，让uWSGI开机自启

uWSGI有一个emperor模式，这种模式下会自动捕捉配置文件的变化，生成新的uWSGI实例

> uWSGI can run in ‘emperor’ mode. In this mode it keeps an eye on a directory of uWSGI config files, and will spawn instances (‘vassals’) for each one it finds.

```bash
# create a directory for the vassals
sudo mkdir /etc/uwsgi
sudo mkdir /etc/uwsgi/vassals
```
vassals目录下可以放置多个配置文件，uWSGI启动时将全部加载

> ~~ps:如果实在不清楚uid, gid的话写root也不是不可以~~

```bash
[emperor] is the uwsgi binary in your system PATH ?
TIME STAMP - [emperor] curse the uwsgi instance cc_uwsgi.ini (pid: ####)
TIME STAMP - [emperor] removed uwsgi instance cc_uwsgi.ini
```

> 2019/10/30: 使用www遇到权限问题，建议root就完事了

以守护进程的方式执行

```bash
uwsgi --emperor /etc/uwsgi/vassals --uid root --gid root --daemonize /var/log/uwsgi-emperor.log
```
最后一步，编写脚本，让uWSGI开机自启

官方推荐的做法是新建文件 /etc/rc.local，把上面这行命令放进去

不过好像不是很管用，开机时并不会自启动，还要手动执行一遍
```bash
/etc/rc.local
```

所以最后我在~/.bashrc下添加了执行命令 /etc/rc.local，这样就做到开机自启了。