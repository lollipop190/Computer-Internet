1.基本配置

---

一个路由器R0

两个交换机 **S1** **S2**

4台pc

- vlan 10:
  - pc1、pc3
- vlan 20:
  - pc2、pc4

---

先将S1和S2之间的链路设置为trunk

S1:

```
Switch(config)#hostname S1
S1(config)#interface FastEthernet0/23 
//这个端口通过交叉线和S2相连
S1(config-if)#switchport mode trunk   
```

S2:

```
Switch(config)#hostname S2
S2(config)#interface GigabitEthernet0/1
//这个端口通过交叉线和S1相连
S2(config-if)#switchport mode trunk 
```

然后划分vlan

```
S1(config)#vlan 10
S1(config)#vlan 20
```

将S1的两个端口分别分给 vlan 10 和 vlan 20

```
S1(config)#interface fa 0/1 
//对于S1，fa 0/1连接pc1 fa0/2连接pc2
//对于S2，fa 0/1连接pc3 fa0/2连接pc4
S1(config-if)#switchport mode access 
S1(config-if)#swichport access vlan 10   
S1(config-if)#exit sw1(config)#interface fa 0/2 S1(config-if)#switchport mode access 
S1(config-if)#switchport access vlan 20
```

对S2作同样操作操作相同

然后将S1与路由器的链路设置成trunk

```
S1(config)#interface fa 0/24 
S1(config-if)#switchport mode trunk
```

R0:

给R0与S1连接的链路对应的端口配置两个子网ip

```
Router(config)#hostname R0
R0(config)#interface GigabitEthernet0/2
R0(config-if)#no ip address 
R0(config-if)#no shutdown 
R0(config)#int GigabitEthernet0/2.10    
R0(config-if)#ip address 192.168.10.1 255.255.255.0
//此为pc1 pc3的网关
R0(config)#int GigabitEthernet0/2.20 
R0(config-if)#ip address 192.168.20.1 255.255.255.0
//此为pc2 pc4的网关
```

最后配置pc的ip

pc1: 192.168.10.2

pc2: 192.168.20.2

pc3: 192.168.10.3

pc4: 192.168.20.3

### 2.验证配置

---

pc1 ping pc3

```
C:\>ping 192.168.10.3

Pinging 192.168.10.3 with 32 bytes of data:

Reply from 192.168.10.3: bytes=32 time=1ms TTL=128
Reply from 192.168.10.3: bytes=32 time<1ms TTL=128
Reply from 192.168.10.3: bytes=32 time<1ms TTL=128
Reply from 192.168.10.3: bytes=32 time<1ms TTL=128

Ping statistics for 192.168.10.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

Pc1 ping pc2

```
C:\>ping 192.168.20.2

Pinging 192.168.20.2 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.20.2:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

### 3.命令列表

| 设置Trunk封装类型 | switchport trunk encapsulation [type]//好像没用 |
| ----------------- | ----------------------------------------------- |
| 设置trunk链路     | switchport mode                                 |
| 划分vlan          | vlan [vlan name]                                |
| 将接口划分入vlan  | swichport access vlan [vlan name]               |
| 显示vlan简要信息  | show vlan brief                                 |

