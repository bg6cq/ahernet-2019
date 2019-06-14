## GPG 实验操作


说明:

* 登录时自动设置了ID=xx，xx是用户名的最后2位数字。


### 1. 生成私钥和公钥

注：默认生成4096 bit私钥可能需要30分钟或更久，服务器上已经执行了命令`rngd -r /dev/urandom`，从硬件获取随机数，加快私钥生成速度。

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
您想要用多大的密钥尺寸？(2048)1024    ## 输入1024 或 2048 或 4096
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

上面主密钥ID是AAF20672，请记住，下面会用到。

### 2. 查看私钥和公钥


```
gpg --list-key --fingerprint

gpg --edit-key AAF20672    ## 请用自己的主密钥ID
q
```

显示的密钥功能，S是签名，C是证书，E是加解密，A是认证。

### 3. 导出私钥和公钥

导出密钥并显示。

请用自己的密钥ID替换下面的命令：
```
gpg --armor --export-secret-keys AAF20672 > user${ID}-sec-key.txt
gpg --armor --export AAF20672 > user${ID}-pub-key.txt

more user${ID}*key.txt 
```

### 4. 将公钥上传到 公钥服务器

```
gpg --keyserver pgp.ustc.edu.cn --send-keys AAF20672
```

命令执行后，使用浏览器访问 http://pgp.ustc.edu.cn/ 输入0xAAF20672 ，可以查询到密钥。

### 5. 通过文件交换公钥

GPG正确工作的前提是获取到其他人的公钥。

找同一台机器上的用户，把 user??-pub-key.txt 拷贝到 /tmp/ 目录

```
gpg --import /tmp/user??-pub-key.txt
gpg --list-key
```

可以导入别人的公钥。

### 6. 通过keyserver交换公钥

请询问旁边用户的公钥ID，在他/她已经把公钥上传服务器后，通过公钥服务器可以获取公钥。

如果知道了对方的公钥ID(下面的例子是2BE69E45)，可以执行
```
gpg --keyserver pgp.ustc.edu.cn --recv-keys 2BE69E45
gpg --list-key
```

注意：近期keyserver工作不正常，这种方法很可能失败。


### 6. 信任别人的公钥

通过上述步骤5或6获取的公钥，默认是不信任的。

如果要信任一个公钥，执行

```
gpg --edit-key 2BE69E45
trust
5
y
q
```
信任后的公钥，将来使用时，不会出现不信任的提示。


### 7. 加密一个文件

加密文件时，需要用到接收方（即将来解密方）的公钥。公钥是上述5、6导入并经过7信任的。

假定user00要发送给user01一个加密文件:

编辑文件 `test_to_user01.txt`，是我们要加密的内容：
```
this is a test from user00 to user01
```

加密操作

-r 后面是接收人的标识。

```
gpg -r user01 --armor -e test_to_user01.txt 
```
默认会生成文件 test_to_user01.txt.asc，可以用more 查看文件内容。

### 8. 解密文件

解密时，需要接收方的私钥，所以执行时会提醒输入私钥的密码。

把以上7加密生成的文件交给 接收人（上面例子是user01），比如通过 /tmp/ 目录中转。

接收人执行以下命令解密：
```
gpg -d /tmp/test_to_user01.txt.asc > out.txt
```

out.txt里是最早输入的`this is a test from user00 to user01`。


### 9. 签名一个文件

签名时，需要用到签名放的私钥，所以执行时会提醒输入私钥的密码。

假定user00要对一个文件签名：

编辑文件 `test_sign_user00.txt`，写入：
```
this file is from user00
```

签名操作：
```
gpg -s --armor test_sign_user00.txt
```

签名得到的结果文件是 test_sign_user00.txt.asc

将这个文件拷贝到 /tmp

### 10. 其他用户的验证签名

其他用户只要导入了user00的公钥，都可以验证签名。

```
gpg --verify /tmp/test_sign_user00.txt.asc
gpg -d /tmp/test_sign_user00.txt.asc
```

第一个命令仅仅验证文件是什么时间，谁签名的。

第二个命令还会显示文件的内容。

### 11. 明文签名

上述9生成的签名文件，必须使用gpg处理后才能看到文件内容，无法直观看到之前的文件内容。

`more test_sign_user00.txt.asc`可以看到签名后的内容。

有时候我们希望文件原文可读，这时可以使用明文签名：
```
gpg --clearsign test_sign_user00.txt
more test_sign_user00.txt.asc
```
可以看到这里有原来的文件内容。

其余验证步骤是相同的。
