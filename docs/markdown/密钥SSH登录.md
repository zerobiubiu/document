# 密钥登陆

> **以下操作均在此条件下使用：**设一台Windows为客户端（也以用Linux做客户端，只是操作有点小出入），远程一台Linux服务器。
>
> Windows采用win10专业工作站版 20H2		Linux使用cent os 8.1

## 第一步生成密钥

执行`ssh-keygen -t rsa`命令，无论是cmd还是power shell都可以得到以下结果![image-20210214191444284](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/image-20210214191444284.png)

不需要管干嘛的，按4下回车就可以。

## 第二步查看密钥

执行完第一步后，会在**用户目录下的.ssh文件夹**（*这个文件夹可能是隐藏的，在Linux一定是隐藏的*）里生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）

## 第三步上传公钥

无论什么方式，将id_rsa.pub上传到你想登录的服务器的用户目录下的.ssh目录下，并且改名为authorized_keys。如果是root用户不用赋予权限，如果是非root用户注意管理权限

## 第五步重启SSH服务

在服务器执行`systemctl restart sshd`命令重启ssh服务即可

## 第六步、登录

在客户端使用`ssh -i 私钥的位置（默认就是~/.ssh/id_rsa） 登录用户@服务器IP`命令就可以登录到服务器了