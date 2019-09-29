# 环境

两台机器 192.168.204.131（主）  192.168.204.132（备）

虚拟IP 192.168.204.111



# nginx安装
分别在两台机器安装好Nginx，安装目录都为/usr/local/nginx，并且修改html文件夹下的index.html文件

将131机器改成Welcome to nginx vm4!

将132机器改成Welcome to nginx vm5!

以便Keepalived 访问nginx的时候做区别.

# 配置防火墙
开启80端口

# Keepalived安装
## 下载
http://www.keepalived.org/download.html
这里选择的版本是1.2.18

## 解压&&安装

```
tar -xf keepalived-1.2.18.tar.gz
cd keepalived-1.2.18/
./configure --prefix=/usr/local/keepalived
make && make install
```

## 将 keepalived 安装成 Linux 系统服务
因为没有使用 keepalived 的默认路径安装（默认是/usr/local） ,安装完成之后，需要做一些工作复制默认配置文件到默认路径


```
mkdir /etc/keepalived
cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/
```

复制 keepalived 服务脚本到默认的地址

```
cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/
cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
ln -s /usr/local/keepalived/sbin/keepalived /sbin/
```

设置 keepalived 服务开机启动

```
chkconfig keepalived on
```

## 修改keepalived配置文件（主）

vi /etc/keepalived/keepalived.conf
```
! Configuration File for keepalived

global_defs {
   # 标识本节点的字条串，通常为hostname，但不一定非得是hostname。故障发生时，邮件通知会用到
   router_id vm4
}


  # keepalived 会定时执行脚本并对脚本执行的结果进行分析，动态调整 vrrp_instance 的优先级。如果脚本执行结果为 0，
  #并且 weight 配置的值大于 0，则优先级相应的增加。如果脚本执行结果非 0，并且 weight配置的值小于 0，则优先级相应的减少。
  #其他情况，维持原本配置的优先级，即配置文件中 priority 对应的值。
vrrp_script chk_nginx {
	# 检测 nginx 状态的脚本路径
	script "/etc/keepalived/nginx_check.sh"
	# 检测时间间隔
	interval 2
	# 如果条件成立，权重-20
	weight -20
}

# 定义虚拟路由， VI_1 为虚拟路由的标示符，自己定义名称
vrrp_instance VI_1 {
	# 主节点为 MASTER， 对应的备份节点为 BACKUP
    state MASTER
	# 绑定虚拟 IP 的网络接口，与本机 IP 地址所在的网络接口相同， 通过ifconfig查看
    interface ens33
	# 虚拟路由的 ID 号， 两个节点设置必须一样， 可选IP最后一段使用, 相同的 VRID 为一个组，他将决定多播的 MAC 地址
    virtual_router_id 31
	# 本机 IP 地址
	mcast_src_ip 192.168.204.131
	# 节点优先级， 值范围 0-254， MASTER 要比 BACKUP 高
    priority 100
	# 优先级高的设置 nopreempt 解决异常恢复后再次抢占的问题
	nopreempt
	# 组播信息发送间隔，两个节点设置必须一样， 默认 1秒
    advert_int 1
	# 设置验证信息，两个节点必须一致
    authentication {
		# 验证方式（这里选择密码）
        auth_type PASS
		# 密码值
        auth_pass 1111
    }
	
	# 将 track_script 块加入 instance 配置块
	track_script {
		# 执行 Nginx 监控的服务
		chk_nginx
	}
	
	# 虚拟 IP 池, 两个节点设置必须一样
    virtual_ipaddress {
		# 虚拟 ip，可以定义多个
        192.168.204.111
    }
}
```

## 修改keepalived配置文件（备）

```
! Configuration File for keepalived

global_defs {
   router_id vm5
}


vrrp_script chk_nginx {
	script "/etc/keepalived/nginx_check.sh"
	interval 2
	weight -20
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 31
	mcast_src_ip 192.168.204.132
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
	track_script {
		chk_nginx
	}
    virtual_ipaddress {
        192.168.204.111
    }
}
```

## 编写 Nginx 状态检测脚本
编写 Nginx 状态检测脚本 /etc/keepalived/nginx_check.sh 

脚本要求：如果 nginx 停止运行，尝试启动，如果无法启动则杀死本机的 keepalived 进程


vi /etc/keepalived/nginx_check.sh
```
#!/bin/bash
A=`ps -C nginx –no-header |wc -l`
if [ $A -eq 0 ];then
/usr/local/nginx/sbin/nginx
sleep 2
if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
	killall keepalived
fi
fi
```
保存后赋予执行权限

```
chmod +x /etc/keepalived/nginx_check.sh
```


## 启动 Keepalived

```
service keepalived start
```

# 测试
1 先将两个机器Nginx都启动

2 先启动从节点keepalived，接着启动主节点keepalived

3 访问虚拟IP 192.168.204.111

4 可以看到请求到的是主节点的Nginx

5 关闭主节点keepalived （service keepalived stop）

6 在访问虚拟IP 192.168.204.111 可以看到请求的是从节点的Nginx

7 重启主节点Keepalived之后，在请求虚拟IP 192.168.204.111 ， 发现又回到了主节点


# 附录keepalived默认配置文件

```
! Configuration File for keepalived

#主要是配置故障发生时的通知对象以及机器标识
global_defs {
   #故障发生时给谁发邮件通知
   notification_email {  
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   #通知邮件从哪个地址发出
   notification_email_from Alexandre.Cassen@firewall.loc
   #通知邮件的smtp地址
   smtp_server 192.168.200.1
   #连接smtp服务器的超时时间
   smtp_connect_timeout 30
   #标识本节点的字条串，通常为hostname，但不一定非得是hostname。故障发生时，邮件通知会用到
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.200.16
        192.168.200.17
        192.168.200.18
    }
}

virtual_server 192.168.200.100 443 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    nat_mask 255.255.255.0
    persistence_timeout 50
    protocol TCP

    real_server 192.168.201.100 443 {
        weight 1
        SSL_GET {
            url {
              path /
              digest ff20ad2481f97b1754ef3e12ecd3a9cc
            }
            url {
              path /mrtg/
              digest 9b3a0c85a887a256d6939da88aabd8cd
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.2 1358 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    sorry_server 192.168.200.200 1358

    real_server 192.168.200.2 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.3 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.3 1358 {
    delay_loop 3
    lb_algo rr
    lb_kind NAT
    nat_mask 255.255.255.0
    persistence_timeout 50
    protocol TCP

    real_server 192.168.200.4 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.5 1358 {
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url {
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

```

# 附录另一个检测脚本

```
#!/bin/bash
run=`ps -C nginx --no-header | wc -l`
if [ $run -eq 0 ]
then
/usr/local/nginx/sbin/nginx -s stop
/usr/local/nginx/sbin/nginx
sleep 3
if [ `ps -C nginx --no-header | wc -l` ]
then
killall keepalived
fi
fi
```


