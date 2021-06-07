## 1. 基本配置

R1，R2两个路由器通过s2/0互联，用Serial DTE线（红色线）

- R1操作：

```
Router(config)#hostname R1
R1(config)#interface Serial2/0
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shutdown

```

- R2操作：

```
Router(config)#hostname R2
R2(config)#interface Serial2/0
R2(config-if)#ip address 192.168.1.2 255.255.255.0
R2(config-if)#no shutdown

```

## 2. 验证3层连通性

```
R2#ping 192.168.1.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 21/23/25 ms
R2#
```

## 3. 使用扩展的ACL封杀R1到R2的ping命令

### 3.1 在R2上设置并应用ACL（这里用的是扩展ACL）

```
R2(config)#access-list 100 deny icmp host 192.168.1.1 host 192.168.1.2
R2(config)#access-list 100 permit icmp any any
R2(config-if)#ip access-group 100 in
```

### 3.2 查看ACL

```
R2#show ip access-lists 
Extended IP access list 100
    10 deny icmp host 192.168.1.1 host 192.168.1.2
    20 permit icmp any any

R2#
```

### 3.3 检测效果

```
R1#ping 192.168.1.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
UUUUU
Success rate is 0 percent (0/5)
```

```
R2#show ip access-lists 
Extended IP access list 100
    10 deny icmp host 192.168.1.1 host 192.168.1.2 (5 match(es))
    20 permit icmp any any
```

## 4. 实验命令列表

|                                                          |                                                              |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| 配置 access list                                         | access-list [list number] [permit\|deny] [source address] [address] [wildcard mask] [log] |
| 将指定访问列表应用到相关接口，<br/>并指定 ACL 作用的方向 | ip access-group {[access-list-number]\|[name]} [in\|out]     |
| 显示已设置的访问控制列表内容                             | show ip access-lists                                         |

## 5. 注意事项

- 对于 ACL 的放置位置，有以下的原则：扩展 ACL 放置在靠近源的位置，标准 ACL放置在靠近目的位置。

- 对于 ACL，有个非常重要的特性，他不能过滤本地数据流！也就是说，对于 R1 上发送的数据，设置在 R1 接口上的 ACL 并不能对它进行过滤。为了能对数据流进行过滤，需要把 ACL 设置在对端的 R2 上 。

