---
layout: post
title:  "centos 7安装gdal"
categories: 
- 部署运维
- 软件安装
tags:  gdal 
author: liuyu
date: 2018/05/18 20:46:25
---
gdal操作tiff的性能，要比geotools快很多。
想要在java中使用gdal，windows可以直接下载到编译好的dll文件([点这里下载][1])，并把存放dll的文件夹配置到path环境变量里即可，有时需要重启一下ide才能识别到
centos下需要自己配置环境和编译
下面以centos7上安装2.2.3版本的gdal为例，介绍整个安装过程

*参考资料
https://www.cnblogs.com/gnool/p/7277474.html
https://blog.csdn.net/hqfgiser/article/details/78710856*

首先，需要下载以下文件，方便起见，已全部上传至网盘:[gdal_all_files][2]
```
gdal223.zip
geos-3.6.2.tar.bz2
hdf-4.2.13.tar.gz
hdf5-1.10.1.tar.gz
jpegsrc.v9b.tar.gz
netcdf-c-4.4.1.1.tar.gz
proj-4.9.3.tar.gz
swig-3.0.12.tar.gz
```
# 安装gdal
## 1.切换到root用户
su - root


## 2.安装依赖环境

yum  -y  groupinstall "Development tools"   #这步不成功貌似也没问题-_-

yum -y install zlib-devel



## 3.分别安装上述安装包如下：
### proj
```
tar -zxvf proj-4.9.3.tar.gz
cd proj-4.9.3
./configure
make
make install
cd ..
```
proj —help 检验安装
### geos
```
tar -zxvf geos-3.6.2.tar.gz
cd geos-3.6.2
./configure
make
make install
ldconfig
cd ..
```
### jpeg
```
tar -zxf jpegsrc.v9b.tar.gz
cd jpeg-9b/
./configure --prefix=/opt/jpeg
make 
make install
cd ..
```
### hdf5
```
tar -zxf hdf5-1.10.1.tar.gz
cd hdf5-1.10.1
export F9X=ifort
./configure --prefix=/opt/hdf5 --with-hdf4=/opt/hdf4 --with-jpeg=/opt/jpeg --enable-java --enable-cxx
make 
make install
cd ..
```
```
tar -zxf hdf-4.2.13.tar.gz
cd hdf-4.2.13
./configure --prefix=/opt/hdf4 --enable-netcdf --enable-jpeg --with-jpeg=/opt/jpeg --enable-hdf5 --with-hdf5=/opt/hdf5 --enable-shared --disable-fortran --enable-java
make
make install
cd ..
```
### netcdf
```
tar -zxf netcdf-4.4.1.tar.gz
cd netcdf-4.4.1
CPPFLAGS="-l/opt/hdf4/include -l/opt/hdf5/include -l/opt/jpeg/include"
LDFLAGS="-l/opt/hdf4/lib -l/opt/hdf5/lib -l/opt/jpeg/lib"
./configure --prefix=/opt/netcdf --enable-hdf5 --with-hdf5=/opt/hdf5 --enable-hdf4 --with-hdf4=/opt/hdf4 --enable-jpeg --with-jpeg=/opt/jpeg --disable-netcdf-4
make
make install
cd ..
```
#### gdal
```
unzip gdal233.zip
cd gdal-2.2.3
./configure --prefix=/opt/gdal --enable-netcdf --with-netcdf=/opt/netcdf --enable-hdf5 --with-hdf5=/opt/hdf5 --enable-hdf4 --with-hdf4=/opt/hdf4
make
make install
cd ..
```
gdalinfo 验证一下

### swig

yum install pcre
yum install pcre-devel
```
tar -zxf swig-3.0.12.tar.gz
cd swig-3.0.12
./configure
make
make install
```
检验安装swig -help

## 4.修改配置文件
vi  /etc/profile
加入
```
export PATH=${PATH}:/opt/hdf4/include:/opt/hdf4/bin:/opt/hdf5/include:/opt/hdf5/bin:/opt/netcdf/include:/opt/netcdf/bin:/opt/gdal/include:/opt/gdal/bin
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/opt/hdf4/lib:/opt/hdf5/lib:/opt/netcdf/lib:/opt/gdal/lib
```
source  /etc/profile


# 编译gdal-java
## 1、安装jdk
略
## 2、编译jar和jni文件
进入<gdal-dir>\swig\java修改java.opt文件，指定jdk的路径，<gdal-dir>为gdal文件夹。内容如下： 
```
JAVA_HOME = /opt/jdk1.8
JAVADOC=$(JAVA_HOME)/bin/javadoc 
JAVAC=$(JAVA_HOME)/bin/javac 
JAVA=$(JAVA_HOME)/bin/java 
JAR=$(JAVA_HOME)/bin/jar 
JAVA_INCLUDE=-I$(JAVA_HOME)/include -I$(JAVA_HOME)/include/linux 
```
编译需要ant支持，所以:
```
yum install ant
```

编译,并将so文件拷贝出来
```
cd <gdal>/swig/java/
make
cd .libs/

cp *.so /usr/local/gdalso/
```
同时，目录下会生成gdal.jar，拷贝到项目中就可以用了，需要注意的是，这个jar在win和linux下并不通用

vi ~/.bashrc ,加上环境变量
```
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/gdalso
```
使环境变量生效，source ~/.bashrc

完成了，享受急速的栅格数据处理性能吧~


  [1]: http://trac.osgeo.org/gdal/wiki/DownloadingGdalBinaries
  [2]: https://pan.baidu.com/s/1ulNu3QqUKMYUpsbnrIfo0w
