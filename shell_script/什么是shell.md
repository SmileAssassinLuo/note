#### shell

##### 一，shell介绍

shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。

Shell 是指一种应用程序，这个应用程序提供了一个界面，**用户通过这个界面访问操作系统内核的服务**。

##### 二，shell脚本

**shell脚本就是将完成一个任务的所有命令按照执行的先后顺序，自上而下的写入到文本文件中，然后给与执行权限。**

###### (1)如何书写shell脚本

新建以`.sh`结尾的文件,脚本组成：

* 脚本开头必须通过`#!bin/bash`指定执行环境,两种方式：
  * `#!user/bin/env bash | python | perl`  
  * `#!user/bin/bash`

* 添加注释 ，**`#`开头是注释，`#!`是特例。**
* 最好加上描述说明，作者，时间，版本号等。
* 脚本内容。

例如，下载安装`nginx`：

`download_install_nginx.sh`:

```shell
#!user/bin/bash
#Author:
#CreateTime:
#Relase Version:
#Description:
yum -y install wget gcc pcre-devel zlib-devel
wget http://nginx.org/download/nginx-1.16.0.tar.gz tar xf nginx-1.16.0.tar.gz cd nginx-1.16.8
./configure --prefix=/usr/local/nginx 
make -j 4
make install
```

###### (2) 执行脚本

* 给执行权限。 `chmod u+x filename`,比如，`chmod 644 nginx.sh`给权限后，执行 `./nginx.sh`; 命令`ll`查看权限。

* 解释器直接运行，不需要给权限。`bqash filename`

  

######  (3)shell变量



