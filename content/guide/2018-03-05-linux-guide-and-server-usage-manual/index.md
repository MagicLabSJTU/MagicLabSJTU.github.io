---
title: linux入门与服务器使用指南
date: 2018-03-05
---

实践中，模型的训练往往需要在带GPU的服务器上进行，如果对linux服务器接触不深，可以参考以下学习资料。

<!--more-->

- **本教程并非一个从零开始或者完全指南，你需要以下能力:**

  - 了解linux 文件结构

  - 能使用命令行在各个路径下自由切换，查看文件夹和文件属性等

  - 了解什么是环境变量. 

  - 了解两个重要的文件的作用： /etc/profile, ~/.bashrc

  - 使用过pip安装package,在自己代码中使用过自己安装的package


- **看完本教程你将会掌握一下能力**

  - 在无root权限的情况下自由的使用服务器

  - 自己编译软件，拥有不google也可以解决依赖各种不对应的问题的能力。

  - 使用conda管理自己的环境，管理自己的依赖包

------

- 任何包都可以**不使用root安装**(不考虑与系统密切相关的包)。
- 没有完整看过[Conda官方参考文档](https://conda.io/docs/user-guide/index.html) 必须阅读第四章。
- 需要自己编译安装软件的,比如caffe，py-faster-rcnn等，必须阅读第三章。

**内容较多，请按需阅读。**

{{< toc >}}

## 1. 常用命令

### 1.0 必须掌握

#### 1.0.1 命令

具体使用方法自行百度或Google。

-  `cd, ls, mkdir`
-  `find` 配合grep 查找文件很方便
- `locate` 直接定位文件位置,速度极快，系统将所有文件建立了一个索引，所以新增的文件需要一段时间才可以locate到. 如`locate libhdf5.so`
- `which` 查找命令的完整路径. 如`which top`会输出`/usr/bin/top`
-  `grep` 可以过滤输出流的文本
-  `top` 查看正在运行的进程
-  `nvidia-smi` 查看显卡占用情况
-  `df -h` 查看磁盘挂载和已用情况
-  `screen` 或者`tmux` 在这些窗口中运行的任务，即使关掉你的终端也不会被杀掉，否则一旦ssh连接关闭任务会被杀死。

#### 1.0.2 环境变量

`/etc/profile`(root才有写权限)和`~/.bashrc`是每次通过`ssh`登陆服务器都会运行的脚本。运行顺序是 `/etc/profile`->`~/.bashrc`。
一些希望每次登陆后都生效的环境变量可以放在`~/.bashrc`里面。

- `PATH`: 额外的可执行文件搜索路径
- `CPATH`: 额外的头文件搜索路径
- `LD_LIBRARY_PATH`: 额外的RunTime 搜索路径
- 上面的路径都会添加到系统标准路径之前。 可以`vim /etc/profile`查看示例。

### 1.1 翻墙

`proxychains4 your-command`比如`proxychains4 curl www.google.com`服务器上翻墙免费提供，**如果想其他平台linux/macOs/windows/ios/android等翻墙，欢迎赞助**。

### 1.2 权限

`ll`, `chmod`
[鸟哥的linux私房菜-第六章linux的文件权限与目录配置](http://cn.linux.vbird.org/linux_basic/0210filepermission.php)

### 1.3 访问服务器

#### 1.3.1 Windows

安装[MobaXterm](https://mobaxterm.mobatek.net/), 自带x11图形转发，使用会产生图形界面的命令就像本地显示屏一样弹出图形界面。

#### 1.3.2 Linux, Mac

命令行`ssh -p port username@host-ip`, 推荐添加公钥访问[使用ssh公钥实现免密访问](https://www.cnblogs.com/Percy_Lee/p/5698603.html)。Mac 需要**安装X11**才能进行x11转发，Ubuntu自带。

***uix平台建议添加config文件，这样登陆远程服务器不需要冗长的命令。**

添加文件`~/.ssh/config`,添加内容类似下面

```null
HOST ss2
    HostName wsl2
    #or ip
    #HostName 192.168.1.140
    Port 10010
    User klaus
    ServerAliveInterval 30
HOST ss1
    HostName wsl1
    Port 10086
    User klaus
    ServerAliveInterval 30
```

添加以后命令行输入`ssh ss1`即可登陆wsl1，账号是klaus。使用`ssh other-username@ss1`则以other-username登陆。

### 1.4 服务器间传输文件

#### 1.4.1 从远程服务器拉到本地

- 单个文件推荐 `scp [-r] -P port username@host-ip:/path/to/your/file /save/to/here`
- 大量文件推荐，可断点续传 `sync -avz -P -e "ssh -p $portNumber" user@remoteip:/path/to/files/ /local/path/`

#### 1.4.2 从本地推到远程服务器

- 单个文件推荐 `scp [-r] -P port /your/data/path username@host-ip:/path/to/save`
- 大量文件推荐，可端点续传 `rsync -avz -P -e "ssh -p $portNumber" /local/path/ user@remoteip:/path/to/files/`

**在`wsl1(192.168.1.139)`和`wsl2(192.168.1.140)`之间传输数据，ip地址请写内网ip，或者写`wsl1`或者`wsl2`，服务器会自动解析到对应ip。内网间传输文件可以达到100MB/S**



## 2. 安装软件

linux下一般称软件为包(package)。安装方法有包管理器安装和手动编译。安装以后的东西主要分为三个部分:

- 可执行文件. **一般保存在`/\**/bin`下**
- 头文件. c/c++的头文件，一般是作为别的package的依赖。**一般保存在`/\**/include`下**
- 链接库. 动态链接库.so和静态链接库.a. 一般作为别的package的依赖.**一般保存在`/\**/lib`或者`/\**/lib64`下**



## 3.头文件和链接库

**头文件**一般是c/c++的头文件,后缀名为`.h,.hxx,.hpp`等.
**链接库**分为动态链接库和静态链接库,linux上后缀名分别为`.so`和`.a`,实质上是**一堆编译后的c/cpp文件(.o)打包成.so或.a**。

**静态链接库.a会被嵌入到编译的程序中，但是动态链接库不会，需要在运行时找到.so文件才能运行。**

一般的软件都会依赖很多package(头文件和链接库)。编译软件时称为**Compile Time**, 运行软件时称为**Run Time**.在 **Compile Time**需要让编译器找到它所依赖的头文件和链接库，在**Run Time**需要让可执行文件找到它所以来的动态链接库。**Compile Time**和**Run Time**寻找依赖的路径是**不一样**的，也就是说**即时编译通过，在运行的时候也会存在找不到动态链接库的情况，一般会报`no defined reference to ...`错误**。

### 3.1 Compile Time

在我们服务器上,查看头文件和链接库的`search path`, run `cpp -v /dev/null -o /dev/null`。编译器会根据`search path`从头一个一个的查找对应的头文件和链接库。也就是说如果安装了同一个package的不同版本，出现在前面的会覆盖掉后面的。

```null
#include <...> search starts here:
 /usr/local/cudnn-6.0/include
 /usr/lib/gcc/x86_64-redhat-linux/4.8.5/include
 /usr/local/include
 /usr/include
LIBRARY_PATH=/usr/lib/gcc/x86_64-redhat-linux/4.8.5/:
/usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../../lib64/:
/lib/../lib64/:
/usr/lib/../lib64/:
/usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../:
/lib/:
/usr/lib/
```

上面的目录分别是编译器寻找头文件和链接库的`search path`，寻找顺序按照路径先后顺序。

#### 3.1.1 **那如果我想要编译的包的依赖不在上面的路径中怎么办？**

有两种方法:

1. **编译选项添加gcc/g++的Flags, `-I`, `-L`。**

   `-I`添加寻找头文件的目录，`-L`添加寻找链接库的目录，这两个flag可以添加多个。

   如`g++ test.cpp -lmylib -L/path/to/your/lib/dir -I/path/to/your/include/dir -I/path/to/your/include/dir2`，这一行命令告诉编译器去`/path/to/your/include/dir`和`/path/to/your/include/dir2`目录下找头文件，去`/path/to/your/lib/dir`找链接库，如果没找到就会去系统标准的`search path`中去找。`-l`告诉编译器去寻找的动态链接库的名字，如`-lmylib`会去寻找名称为`libmylib.so`或者`libmylib.a`链接库`(lib + mylib + .so/.a)`。需要寻找的头文件的名称已在源代码的`#include ""`中指定了。

2. **添加环境变量 `CPATH`和`LIBRARY_PATH`**
   `CPATH`是指定额外头文件的`search path`，多个目录以冒号`:`分隔。
   `LIBRARY_PATH`是指定额外lib的`search path`,多个目录以冒号`:`分隔。**特别需要注意的是，在64位系统上，每一个添加的目录，系统默认编译的gcc/g++会添加两个目录，`${your-dir}/../lib64`和`${your-dir}`。并且，对于所有目录`lib64`会位于`lib`之前,就是说所有`${any-prefix}/lib`目录都会位于所有`${any-prefix}/lib64`之后。**
   `CPATH`和`LIBRARY_PATH`会添加到`-I,-L`添加的目录之后，系统标准目录之前。
   例如添加`/usr/local/lib`

```null
export LIBRARY_PATH=/usr/local/lib
#如果LIBRARY_PATH不为空, 命令行输入echo $LIBRARY_PATH 打印不为空
#加上一个:$LIBRARY_PATH会在原有$LIBRARY_PATH前面再加上你想加的路径
export LIBRARY_PATH=/usr/local/lib:$LIBRARY_PATH
```

添加之后`search path`变为

```null
LIBRARY_PATH=/usr/local/lib/../lib64/:
/usr/lib/gcc/x86_64-redhat-linux/4.8.5/:
/usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../../lib64/:
/lib/../lib64/:
/usr/lib/../lib64/:
/usr/local/lib/:
/usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../:
/lib/:
/usr/lib/

#/usr/local/lib64排在第一位，但是/usr/local/lib在/lib64,/usr/lib64之后
```

#### 3.1.2 SEARCH PATH order

现在我们有`-I,-L`,`CPATH, LIBRARY_PATH`两种方式添加额外的path，还有一个后面会讲的`LD_LIBRARY_PATH`，这三种方法都会对编译器的search path产生影响。
假设

```null
LD_LIBRARY_PATH=dirlist
ld ... -Lpath1 ... -Lpathn ...
LIBRARY_PATH=libpathlist
```

那么search path 顺序为:

```null
# -L > LD_LIBRARY_PATH > LIBRARY_PATH
path1 ... pathn dirlist libpathlist
```

### 3.2 Run Time

Run Time 即为程序运行加载的时候。Compile Time中`-L`和`LIBRARY_PATH`不会对Run Time 产生任何影响。也就是说，在运行程序的时候`-L`或者`LIBRARY_PATH`指定的额外路径如果没有被添加到runtime的search path里去，那么程序将会找不到对应的lib！

#### 3.2.1 Run Time Search Path Order

1. rpath. 编译时指定的路径，如`g++ -o hello hello.c -Wl,-rpath,/usr/local/lib/hello/B,-rpath,/usr/local/lib/hello/A`

2. LD_LIBRARY_PATH. 该环境变量中指定的路径。**它的一个副作用是它的路径会影响编译器编译的路径，见3.1.2**

3. /etc/ld.so.conf 中指定的路径. 通过`ldconfig -v 2&gt;/dev/null | grep -v ^$'\t'`查看

   ```null
   (klaus_all) [klaus@wsl1 build]$ ldconfig -v 2&gt;/dev/null | grep -v ^$'\t'
   /usr/lib64/dyninst:
   /usr/lib64/iscsi:
   /usr/lib64/mysql:
   /usr/lib64/qt-3.3/lib:
   /lib:
   /lib64:
   /lib/sse2: (hwcap: 0x0000000004000000)
   /lib64/sse2: (hwcap: 0x0000000004000000)
   /lib64/tls: (hwcap: 0x8000000000000000)
   ```
   
4. /lib64,/usr/lib64,/lib,/usr/lib. 默认路径。


一般的程序的rpath只有当前程序所在目录，所以如果你依赖了3和4中没有的路径的,你需要额外指定`LD_LIBRARY_PATH`, 比如 `export LD_LIBRARY_PATH=/some/path:$LD_LIBRARY_PATH`,为了方便可以将这一句添加到`~/.bashrc`中。

### 3.3 有用的命令

弄清楚了Compile Time 和Run Time 的search path，在编译程序，运行程序中遇到的问题基本可以自己解决了。有几个常用的命令可以帮助定位问题.

- ldd.

查看一个lib或者executable所依赖的.so和其link的路径。如

```null
(klaus_all) [klaus@wsl1 build]$ ldd libextract_dct.so
    linux-vdso.so.1 =>  (0x00007ffd9876c000)
    libjpeg.so.9 => /home/klaus/.conda/envs/klaus_all/lib/libjpeg.so.9 (0x00007fe23df50000)
    libc.so.6 => /lib64/libc.so.6 (0x00007fe23db72000)
    /lib64/ld-linux-x86-64.so.2 (0x00007fe23e38f000)
```

- nm.

列出一个lib或者executable定义的所有symbol.如

```null
(klaus_all) [klaus@wsl1 build]$ nm libextract_dct.so
0000000000202088 B __bss_start
0000000000202088 b completed.6337
                 w __cxa_finalize@@GLIBC_2.2.5
...skipped...
                 U jpeg_finish_decompress@@LIBJPEG_9.0
                 U jpeg_read_coefficients@@LIBJPEG_9.0
                 U jpeg_read_header@@LIBJPEG_9.0
                 U jpeg_save_markers@@LIBJPEG_9.0
                 U jpeg_std_error@@LIBJPEG_9.0
                 U jpeg_stdio_src@@LIBJPEG_9.0
                 w _Jv_RegisterClasses
                 U malloc@@GLIBC_2.2.5
0000000000000b18 T read_jpeg
0000000000000a60 t register_tm_clones
0000000000202088 d __TMC_END__
```

- readelf -d.

列出ELF文件的信息,可以用来查看rpath. 如

```null
(klaus_all) [klaus@wsl1 build]$ readelf -d libextract_dct.so

Dynamic section at offset 0x1de8 contains 27 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libjpeg.so.9]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x000000000000000e (SONAME)             Library soname: [libextract_dct.so]
 0x000000000000000f (RPATH)              Library rpath: [/home/klaus/.conda/envs/klaus_all/lib/libjpeg.so:/home/klaus/.conda/envs/klaus_all/lib]
 0x000000000000000c (INIT)               0x918
...skipped...
 0x000000006ffffff9 (RELACOUNT)          3
 0x0000000000000000 (NULL)               0x0
```



## 4. conda环境管理

[conda](https://conda.io/docs/user-guide/tasks/manage-environments.html)是python的一个包管理器，但是它**不仅仅是**一个python的包管理器。

- 环境管理。你可以创建任意多的环境，每个环境可以安装不同的包，不同的版本，每个环境之间互相不影响。
- **它可以安装非python包**,如`tmux`, `cmake`, `hdf5`等等.需要什么可以先去google一下,比如搜索"tmux conda"，或者直接去[Anaconda Cloud](http://202.120.36.7:40222/61/anaconda.org)搜索是否有对应的包。
- 兼容pip. **可以使用pip安装python包而不需要root权限。**推荐首先查看conda上是否有这个包，如果没有再使用pip安装。

### 4.1 管理环境

服务器上配置好了几个环境:
\- `root`. 安装好miniconda后干净的环境，仅含有必要的几个包。
\- `common`. 常用的几个python包，编译caffe所有需要的依赖也在common中装好了。没有安装任何框架。
\- `dl-2.7`. 装好了keras, tensorflow, pytorch等深度学习框架。python版本为2.7

这两个环境只有`root`用户有权限更改（安装,删除package）,但是所有用户都有使用权限(read)。

### 4.2 使用环境

- 查看环境. `conda env list`. 查看所有的环境,其它用户的环境不可见。
- 查看当前环境中安装的包. `conda list`. 加上`grep`可以
- 激活环境. `source activate env-name`. env-name为环境名称如common, dl-2.7, 下同。
- 停止环境. `source deactivate env-name`

如果某个环境会经常

### 4.3 创建环境

- `conda create -n env-name` 创建一个名为env-name的环境，
- `conda create -n env-name numpy scipy` 创建一个包含numpy和scipy包的环境
- `conda create -n env-name scipy=0.15.0` 指定package的版本
- `conda create -n env-name --clone cloned-env` 克隆cloned-env到env-name中，所有cloned-env中的包都会被克隆岛env-name中。

**自己创建的环境有写权限，你可以对它做任何事情**

### 4.4 安装卸载package

- conda install package-name 安装
- conda remove package-name 卸载

conda 自带了几个default channel(类似仓库，每个仓库里有很多package), 但有很多额外的channel，使用上面安装命令找不到的包在其它channel可能会找到。可以去google `package-name conda` 或者[Anaconda Cloud](http://202.120.36.7:40222/61/anaconda.org)搜索是否存在。

### 4.5 目录结构

`root`创建的环境都会被安装到`$CONDA_INSTALL_PATH/envs`下，其它用户创建的环境会被安装到`~/.conda/envs`下.

每个环境下有三个重要的目录:
\- bin
\- include
\- lib

这对应了第二章的三部分。使用conda安装的bin会默认添加到`PATH`里去(可以在命令行直接运行)。**include 和 lib文件夹下的东西不会添加到CompileTime/RunTime 的 search path里去**，所以你需要编译运行什么软件可以将这两个目录分别添加到`CPATH`和`LD_LIBRARY_PATH`里去。