
**使用方法**

------

使用root用户登录，运行以下命令：

```shell
wget -N  --no-check-certificate  https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && bash bbr.sh
```

输入命令后，回车，会提示你按任意键继续安装。

重要提示：安装过程中，看着走不动其实是在下载东西，不要按任何键，否则最后会安装不成功。 安装按完成后会提示你输入Y重启，输入Y回车即可。如果你在安装过程中按了回车键，最后手动重启下服务器也可以。

 重启完成后，进入 VPS，验证一下是否成功安装最新内核并开启 TCP BBR，输入以下命令：

```shell
uname -r
```

查看内核版本，含有 4.1x 就表示 OK 了

```shell
sysctl net.ipv4.tcp_available_congestion_control
```

返回值一般为：

 net.ipv4.tcp_available_congestion_control = bbr cubic reno



```shell
sysctl net.ipv4.tcp_congestion_control
```

返回值一般为：

 net.ipv4.tcp_congestion_control = bbr



```shell
sysctl net.core.default_qdisc
```

返回值一般为：

 net.core.default_qdisc = fq



```shell
lsmod | grep bbr
```

返回值有 tcp_bbr 模块即说明bbr已启动。

**内核升级方法**

------

如果是 CentOS 系统，执行如下命令即可升级内核：

```shell
yum --enablerepo=elrepo-kernel -y install kernel-ml kernel-ml-devel
```

CentOS 6 的话，执行命令：

```shell
sed -i 's/^default=.*/default=0/g' /boot/grub/grub.conf
```

CentOS 7 的话，执行命令：

```shell
grub2-set-default 0
```

如果是 Debian/Ubuntu 系统，则需要手动下载最新版内核来安装升级。
 去[这里](http://kernel.ubuntu.com/~kernel-ppa/mainline/)下载最新版的内核 deb 安装包。
 如果系统是 64 位，则下载 amd64 的 linux-image 中含有 generic 这个 deb 包；
 如果系统是 32 位，则下载 i386 的 linux-image 中含有 generic 这个 deb 包；
 安装的命令如下（以最新版的 64 位 4.12.4 举例而已，请替换为下载好的 deb 包）：

```shell
dpkg -i linux-image-4.12.4-041204-generic_4.12.4-041204.201707271932_amd64.deb
```

安装完成后，再执行命令：

```shell
/usr/sbin/update-grub
```

最后，重启 VPS 即可。

**特别说明**

------

1、如果你使用的是 Google Cloud Platform （GCP）更换内核，有时会遇到重启后，整个磁盘变为只读的情况。只需执行以下命令即可恢复：

```shell
mount -o remount rw /
```

2、如果你使用的是 Linode ，那么无需更换内核，只需修改配置即可开启 BBR：\

```shell
sed -i '/net.core.default_qdisc/d' /etc/sysctl.conf
sed -i '/net.ipv4.tcp_congestion_control/d' /etc/sysctl.conf
echo "net.core.default_qdisc = fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control = bbr" >> /etc/sysctl.conf
sysctl -p
```