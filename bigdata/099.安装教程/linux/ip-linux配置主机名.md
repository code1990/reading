## 2.4 修改主机名(root)

#### 1）修改linux的hosts文件

```shell
## （1）进入Linux系统查看本机的主机名。通过hostname命令查看
hostname
## （2）如果感觉此主机名不合适，我们可以进行修改
vi /etc/sysconfig/network
## 内容如下
NETWORKING=yes

NETWORKING_IPV6=no

HOSTNAME= hadoop101

:wq
## （3）打开/etc/hosts
vim /etc/hosts
## 内容如下
192.168.1.100 hadoop100

192.168.1.101 hadoop101

192.168.1.102 hadoop102

192.168.1.103 hadoop103

192.168.1.104 hadoop104

192.168.1.105 hadoop105

192.168.1.106 hadoop106

192.168.1.107 hadoop107

192.168.1.108 hadoop108

192.168.1.109 hadoop109

192.168.1.110 hadoop110
## （6）并重启设备，重启后，查看主机名，已经修改成功
reboot
hostname
```

#### 2）修改window7的hosts文件

```shell
##（1）进入C:\Windows\System32\drivers\etc路径

##（2）打开hosts文件并添加如下内容

192.168.1.100 hadoop100

192.168.1.101 hadoop101

192.168.1.102 hadoop102

192.168.1.103 hadoop103

192.168.1.104 hadoop104

192.168.1.105 hadoop105

192.168.1.106 hadoop106

192.168.1.107 hadoop107

192.168.1.108 hadoop108

192.168.1.109 hadoop109

192.168.1.110 hadoop110

```

