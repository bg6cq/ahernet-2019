## Docker 实验操作


说明:

* 登录时自动设置了ID=xx，xx是用户名的最后2位数字。
* 设置了alias dc='docker container'，以后任何输入docker container的地方，都可以使用dc。


建议这里下载 [https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) 连接服务器。

### 1. 下载一个最小alpine Linux

```
docker image pull alpine
docker image ls
```

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
docker run --rm -p 90${ID}:80 --name user${ID}nginx -d user${ID}/nginx

```

这时，使用PC机可以访问 http://x.x.x.x:9000 (IP请用服务器IP，9000最后2位用自己编号)，看到

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
docker run --rm -p 90${ID}:80 --name user${ID}ipdesc -d user${ID}/ipdesc

```

这时，使用PC机可以访问 http://x.x.x.x:9000 (IP请用服务器IP，9000最后2位用自己编号)，看到IP地址来源信息。

访问  http://x.x.x.x:9000/y.y.y.y 可以得到y.y.y.y的地址来源信息。

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
docker run -dit --name user${ID}php --rm -v $PWD:/var/www/html -p 90${ID}:80 php:apache
docker cp index.php  user${ID}php:/
```

这时，使用PC机可以访问 http://x.x.x.x:9000 (IP请用服务器IP，9000最后2位用自己编号)，可以看到一个对web服务器的控制界面。

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
docker run -dit --name user${ID}php --rm -v $PWD:/var/www/html:ro -p 90${ID}:80 php:apache
```

这时，使用PC机可以访问 http://x.x.x.x:9000 (IP请用服务器IP，9000最后2位用自己编号)，因为是只读映射，向 /var/www/html 上传文件会出现错误。

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

注意：9000最后2位请用自己编号代替
```
version: '3'
services:

  web:
    build: .
    ports:
     - "9000:5000"  #注意：9000最后2位请用自己编号代替

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

使用PC机可以访问 http://x.x.x.x:9000 (IP请用服务器IP，9000最后2位用自己编号)，看到统计信息刷新一次加1。

测试时，服务器的窗口能看到一些调试信息。

CTRL-C可以终止。

#### 9.7 正式运行

正式运行，可以用命令 docker-compose up -d 在后台执行。

`docker-compose down`可以停止。
