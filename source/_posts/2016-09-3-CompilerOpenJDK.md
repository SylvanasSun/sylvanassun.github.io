---
layout:     post
title:      "在CentOS7下手动编译JDK"
subtitle:   "Compile OpenJDK"
date:       2016-09-03 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - Java
    - JVM
tags:
    - JVM
    - Java
---


### 下载OpenJDK源码

&nbsp;&nbsp;oepnjdk下载地址为 https://jdk7.java.net/source.html

### 安装编译依赖环境

 1. 安装alsa包.
 
    ```
    yum install alsa-lib-devel
    ```
    
 2. 安装cups-devel 

    ```
    yum install cups-devel
    ```
    
 3. 安装X相关库

    ```
    yum install libX*
    ```
    
 4. 安装gcc-c++    

    ```
    yum install gcc gcc-c++
    ```
    
 5. 安装freetype 

    ```
    rpm -ivh freetype-2.4.11-9.el7.x86_64.rpm
    下载地址:http://rpm.pbone.net/index.php3/stat/4/idpl/26641422/dir/centos_7/com/freetype-2.4.11-9.el7.x86_64.rpm.html
    ```
    
 6. 安装Ant

    ```
    tar -zvxf apache-ant-1.9.6-bin.tar.gz
    下载地址:http://ant.apache.org/bindownload.cgi
    ```
    
&nbsp;&nbsp;也可以使用以下命令一次性完成依赖安装:

```
yum -y install build-essential gawk m4
openjkd-6-jkd libasound2-dev libcups2-dev 
libxrender-dev xorg-dev xutils-dev
xllproto-print-dev binutils libmotif3
libmotif-dev ant
```

### 配置环境变量

&nbsp;&nbsp;OpenJDK在编译时需要读取很多环境变量,但大部分都是有默认值的,必须设置的只有两个环境变量: LANG和ALT_BOOTDIR.

&nbsp;&nbsp;**LANG**是用来设定语言选项的,必须设置为: **export LANG=C**.

&nbsp;&nbsp;**ALT_BOOTDIR**则是用来设置Bootstrap JDK的位置,Bootstrap JDK就是一个可用的6u14以上版本的JDK,它是用来编译OpenJDK中部分Java实现的代码的.

&nbsp;&nbsp;另外,如果之前设置了JAVA_HOME和CLASSPATH两个环境变量,在编译之前必须取消,否则在Makefile脚本中检查到有这两个变量存在,会有警告提示.

```
unset JAVA_HOME
unset CLASSPATH
```

&nbsp;&nbsp;其他环境变量如下:

```
#语言选项，这个必须设置，否则编译好后会出现一个HashTable的NPE错
export LANG=C

#Bootstrap JDK的安装路径。必须设置。 
export ALT_BOOTDIR=/Library/Java/JavaVirtualMachines/jdk1.7.0_04.jdk/Contents/Home

#允许自动下载依赖
export ALLOW_DOWNLOADS=true

#并行编译的线程数，设置为和CPU内核数量一致即可
export HOTSPOT_BUILD_JOBS=6
export ALT_PARALLEL_COMPILE_JOBS=6

#比较本次build出来的映像与先前版本的差异。这个对我们来说没有意义，必须设置为false，否则sanity检查会报缺少先前版本JDK的映像。如果有设置dev或者DEV_ONLY=true的话这个不显式设置也行。 
export SKIP_COMPARE_IMAGES=true

#使用预编译头文件，不加这个编译会更慢一些
export USE_PRECOMPILED_HEADER=true

#要编译的内容
export BUILD_LANGTOOLS=true 
#export BUILD_JAXP=false
#export BUILD_JAXWS=false 
#export BUILD_CORBA=false
export BUILD_HOTSPOT=true 
export BUILD_JDK=true

#要编译的版本
#export SKIP_DEBUG_BUILD=false
#export SKIP_FASTDEBUG_BUILD=true
#export DEBUG_NAME=debug

#把它设置为false可以避开javaws和浏览器Java插件之类的部分的build。 
BUILD_DEPLOY=false

#把它设置为false就不会build出安装包。因为安装包里有些奇怪的依赖，但即便不build出它也已经能得到完整的JDK映像，所以还是别build它好了。
BUILD_INSTALL=false

#编译结果所存放的路径
export ALT_OUTPUTDIR=/Users/IcyFenix/Develop/JVM/jdkBuild/openjdk_7u4/build
```

### 编译JDK

 - &nbsp;&nbsp;全部设置好之后,可用在OpenJDK目录下使用 make sanity来检查设置是否正确,如果正确则会会输出：Sanity check passed.

 - &nbsp;&nbsp;之后,输入make命令开始编译,(make不添加参数默认为编译make all).

 - &nbsp;&nbsp;编译完成后,可以将目录复制到JAVA_HOME中,作为一个完整的JDK使用,编译出来的虚拟机,在-version命令中带有用户的机器名.

 - &nbsp;&nbsp;如果只想单独编译HotSpot的话，那么使用hotspot/make目录下的MakeFile进行替换即可，其他参数与前面一致，这时候虚拟机的输出结果存放在build/hotspot/outputdir/linux_amd64_compiler2 中，里面对应了不同的优化级别的目录.

 - &nbsp;&nbsp;在不同机器上,最后一个目录名称会有所差别,bad表示Mac OS系统(内核为FreeBSD),amd64表示是64位JDK(32位为x86),compiler2表示是Server VM(Client VM表示是compiler1).

 - &nbsp;&nbsp;在运行虚拟机前，还要手工编辑目录下的env.sh文件，这个文件由编译脚本自动产生，用于设置虚拟机的环境变量，里面已经发布了”JAVA_HOME，CLASSPATH，HOTSPOT_BUILD_USER” 3个环境变量，还需要增加一个“LD_LIBRARY_CLASSPATH”,内容如下:

   -

   ```
    LD_LIBRARY_PATH=.:${JAVA_HOME}/jre/lib/amd64/native_threads:${JAVA_HOME}/jre/lib/amd64:
    export LD_LIBRARY_PATH
   ```

 - &nbsp;&nbsp;然后执行以下命令启动虚拟机(这时的启动机器名为gamma).

   -

   ```
    . ./env.sh
    ./gamma -version   #有可能是test_gamma,这是自带的一段八皇后代码 
   ```

### Exception

 1. 报错:
    
    ```
     Error: time is more than 10 years from present: 1104530400000 when building java/openjdk* lists.freebsd.org
    ```
    
    通过修改CurrencyData.properties文件, 把10年之前的时间修改为10年之内即可
    Index: /usr/openjdk/jdk/src/share/classes/java/util/CurrencyData.properties把2006改掉就可以重新编译了.
    
### End

> 资料参考于 <<深入理解 Java虚拟机 第二版>>.