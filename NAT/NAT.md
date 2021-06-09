# NAT

![](https://box.nju.edu.cn/f/cda31e25470a4726bbd1/?dl=1)

## 介绍

Router2是外网网段

在Router1上配置NAT来实现Router0和Router2的交流

## 实验步骤

#### 1 配置各自路由器的IP

**Router0**

```
Router(config)#interface S2/0
Router(config-if)#ip address 192.168.1.1 255.255.255.0
Router(config-if)#no shutdown
```

**Router1**

```
Router(config)#int S2/0
Router(config-if)#ip address 192.168.1.2 255.255.255.0
Router(config-if)#int S3/0
Router(config-if)#ip address 200.1.1.1 255.255.255.0
```

**Router3**

```
Router(config)#interface S2/0
Router(config-if)#ip address 200.1.1.2 255.255.255.0
Router(config-if)#no shutdown
```

#### 2 给Router1配置NAT

**Router1**

```Router(config-if)#ip nat inside source static 192.168.1.1 200.1.1.254
Router(config-if)#ip nat inside source static 192.168.1.1 200.1.1.254
Router(config)#int Se 2/0
Router(config-if)#ip nat inside 
Router(config-if)#int S3/0
Router(config-if)#ip nat outside
Router(config-if)#end
```

#### 3 尝试从R0pingR2

**Router0**

```
Router#ping 200.1.1.2
```

发现ping不通

```
Router#show ip nat translations
```

查看NAT转换表也没有地址转换，是因为从R0到200.1.1.0网段需要配一条静态路由

#### 4 增加路由

**Router0**

```
Router(config)#ip route 200.1.1.0 255.255.255.0 S2/0
```

#### 5 再次尝试ping

```
Router#ping 200.1.1.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 200.1.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/16/22 ms
```

ping通了 NAT成功