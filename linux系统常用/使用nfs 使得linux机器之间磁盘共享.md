# 环境说明
centos 7.5 三个节点:
- 192.168.201.131 （主节点）
- 192.168.201.132
- 192.168.201.133

# 服务端（192.168.201.131）
**ps:先安装防火墙**
## 安装nfs
（实际上需要安装两个包nfs-utils和rpcbind, 不过当使用yum安装nfs-utils时会把rpcbind一起安装上）
```
yum install -y nfs-utils   
```
## 创建共享文件夹

```
mkdir -p /home/nfs
```

## 编辑/etc/exports文件，添加从节点

```
/home/nfs 192.168.204.132(rw,sync,fsid=0) 192.168.204.133(rw,sync,fsid=0)
```
## 修改 /etc/sysconfig/nfs 文件
在最后面加上
```
RQUOTAD_PORT="4001"
MOUNTD_PORT="4002"
STATD_PORT="4003"
```
## 修改/etc/modprobe.d/lockd.conf

```
options lockd nlm_tcpport=4004
options lockd nlm_udpport=4004
```
## 在防火墙中加入端口（为两个从节点机器加入端口）

vi /etc/sysconfig/iptables
```
-A INPUT -s 192.168.204.132 -p udp -m udp --dport 2049 -j ACCEPT
-A INPUT -s 192.168.204.132 -p tcp -m tcp --dport 2049 -j ACCEPT
-A INPUT -s 192.168.204.132 -p udp -m udp --dport 111 -j ACCEPT
-A INPUT -s 192.168.204.132 -p tcp -m tcp --dport 111 -j ACCEPT
-A INPUT -s 192.168.204.132 -p tcp -m tcp --dport 4001:4004 -j ACCEPT
-A INPUT -s 192.168.204.132 -p udp -m udp --dport 4001:4004 -j ACCEPT

-A INPUT -s 192.168.204.133 -p udp -m udp --dport 2049 -j ACCEPT
-A INPUT -s 192.168.204.133 -p tcp -m tcp --dport 2049 -j ACCEPT
-A INPUT -s 192.168.204.133 -p udp -m udp --dport 111 -j ACCEPT
-A INPUT -s 192.168.204.133 -p tcp -m tcp --dport 111 -j ACCEPT
-A INPUT -s 192.168.204.133 -p tcp -m tcp --dport 4001:4004 -j ACCEPT
-A INPUT -s 192.168.204.133 -p udp -m udp --dport 4001:4004 -j ACCEPT

```
重启systemctl restart iptables.service


## 为rpcbind和nfs做开机启动

```
systemctl enable rpcbind.service
systemctl enable nfs-server.service
```
## 启动rpcbind和nfs

```
systemctl start rpcbind.service
systemctl start nfs-server.service
```



# 客户端

```
# 安装
yum install -y nfs-utils
# 设置开启启动
systemctl enable rpcbind.service
# 启动服务
systemctl start rpcbind.service
# 创建文件夹
mkdir -p /home/nfs
# 查看挂载
showmount -e 192.168.204.131
# 挂载（将nfs服务器的挂载文件夹，挂载到当前机器的文件夹下）
mount -t nfs 192.168.204.131:/home/nfs /home/nfs

# 设置开机自动挂载
vi /etc/fstab
192.168.204.131:/home/nfs /home/nfs nfs  defaults 0 0

# 移除挂载
umount /home/nfs nfs
```

# 说明
如果主节点挂了，重启之后，从节点需要重新挂载。
先移除挂载，这个命令需要几分钟，然后在重新挂载即可。


