### 环境准备

1. 安装CentOS 7.1810

2. 设置IP地址

3. yum update -y 更新

4. yum install -y git gcc docker glibc-static epel-release
  
yum install -y python-pip

pip install docker-compose

5. systemctl enable docker

6. vi /etc/selinux/config

修改
SELINUX=enforcing 为disabled

7. vi /etc/docker/daemon.json

{
    "live-restore": true,
    "group": "dockerroot"
}

8. vi /etc/bashrc

增加

export ID=$(whoami |cut -c5-6)

7. 

M=1
PASS=pass
for i in `seq 0 9`; do 
   adduser user${M}${i}
   echo ${PASS} | passwd --stdin user${M}${i}
   usermod -aG dockerroot user${M}${i}
done

8. 重新启动，selinux修改才会生效
