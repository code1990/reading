## jdk Linux下环境搭建(root用户)

#### 1)卸载现有jdk

(1)查询是否安装java软件：

```shell
rpm -qa|grep java
```

(2)如果安装的版本低于1.7，卸载该jdk：

```shell
rpm -e 软件包
```

---------

#### 2)上传jdk

alt+p 打开文件上传窗口

```shell
cd /opt/software
```

拖动文件到当前窗口

--------------------

#### 3) 检查文件是否上传成功

```shell
cd software/

ls
```

-----------------

#### 4)解压jdk到/opt/module目录下

```shell
tar -zxf jdk-7u79-linux-x64.gz -C /opt/module/
```

#### 5)配置jdk环境变量

```shell
##(1)先获取jdk路径：如 /opt/module/jdk1.7.0_79  
 pwd   
## (2)打开/etc/profile文件：
 vi /etc/profile
## 在profie文件末尾添加jdk路径：

##JAVA_HOME

export JAVA_HOME=/opt/module/jdk1.7.0_79

export PATH=PATH:JAVA_HOME/bin

## (3)保存后退出：

:wq

## (4)让修改后的文件生效：

source  /etc/profile

## (5)重启(如果java –version可以用就不用重启)：	

 sync

 reboot

```

#### 6)测试jdk安装成功

```shell
 java -version
```

 

拖