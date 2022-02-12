# forever

## 何为forever

> forever可以看做是一个nodejs的守护进程，能够启动，停止，重启我们的app应用。

**官方的说明是说：**

> A simple CLI tool for ensuring that a given script runs continuously (i.e. forever).
> 一个用来持续（或者说永远）运行一个给定脚本的简单的命令行工具

[Github地址](https://github.com/nodejitsu/forever)

## forever用途

forever的用途就是帮我们更好的管理我们node App服务，本质上就是在forever进程之下，创建一个node app的子进程。
比如，你有一个基于express的或者其他的一些个应用那么，它将会很方便你更新和操作你的服务，并且保证你服务能持续运行。
更好的一点就是每次更改文件，它都可以帮你自动重启服务而不需要手动重启。

## 安装forever

记得加-g，forever要求安装到全局环境下

```shell
npm install forever -g
```

## forever使用说明

**简单的启动**

```shell
forever start app.js
```

**启动所有**

```shell
forever restartall
```

**指定forever信息输出文件**

> 当然，默认它会放到~/.forever/forever.log
>
> 或者通过 forever list 能查看到对应的日志。

```shell
forever start -l forever.log app.js
```

**指定app.js中的日志信息和错误日志输出文件**

> -o 就是console.log输出的信息
>
> -e 就是console.error输出的信息

```shell
forever start -o out.log -e err.log app.js
```

**追加日志**

> forever默认是不能覆盖上次的启动日志，所以如果第二次启动不加-a，则会不让运行

```shell
forever start -l forever.log -a app.js
```

**监听当前文件夹下的所有文件改动（文件改动监听并自动重启）**

```shell
forever start -w app.js
```

**显示所有运行的服务**

```shell
forever list
```

**停止所有运行的node App**

```shell
forever stopall
```

**停止其中一个node App**

```shell
forever stop app.js
```

**根据ID停止**

```shell
# 找到对应的id
forever list 

# 停止
forever stop [id]
```

**配置环境变量**

使用unix下的crontab（定时任务）需要注意配置好环境变量。

```shell
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```

 