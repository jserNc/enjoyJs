---
title: https 本地环境搭建(windows)
date: 2016-10-14 16:45:00
tags: method
---


*（注：这里以 www.nc.com 域名为例）*

### 1. 安装支持 ssl 功能的 apache
  
**这里采用版本：**

httpd-2.2.21-win32-x86-openssl-0.9.8r.msi

<!-- more -->

**a) 配置 apache（conf/httpd.conf）使其支持 ssl**

```
LoadModule ssl_module modules/mod_ssl.so
```
```
Include conf/extra/httpd-ssl.conf
```

**b) 修改 conf/extra/httpd-ssl.conf 文件**

```
SSLCertificateFile "D:/WAMP/Apache/conf/server.crt"
```
```
SSLCertificateKeyFile "D:/WAMP/Apache/conf/server.key"
```
```
DocumentRoot "D:/www/www.nc.com"
```

### 2. 为网站生成证书和私钥

**a) 生成私钥server.key**

cmd 进入 Apache/bin 目录，输入：

```
openssl genrsa -out server.key 1024
```

**b) 生成签署申请**

用上一步的密钥生成证书请求文件 server.csr：

```
openssl req -new –out server.csr -key server.key -config ..\conf\openssl.cnf 
```

这里有一系列参数需要输入：

```
Country Name (2 letter code) [AU]: CN　国家代码
```
```
State or Province Name (full name) [Some-State]: SH　省份
```
```
Locality Name (eg, city) []: SH　城市
```
```
Organization Name (eg, company): nc　公司名称
```
```
Organizational Unit Name (eg, section) []: nc　部门名称
```
```
Common Name (eg, YOUR name) []: nc.com　申请证书的域名
```
```
Email Address []: nanc@nc.com　邮箱
```
```
A challenge password []　(可不输入，直接回车)
```
```
An optional company name []　(可不输入，直接回车)
```

*（注：Common Name和httpd.conf中的server name要一致）*

### 3. 通过 CA 为网站服务器签署证书

**a) 生成 CA 私钥**

```
openssl genrsa -out ca.key 1024
```

**b) 利用 CA 的私钥产生 CA 的自签署证书**

```
openssl req -new -x509 -days 365 -key ca.key -out ca.crt -config ..\conf\openssl.cnf
```

*（注：这里需要输入的一系列参数同上，Common Name为服务器域名，如果在本机，为本机IP）*

**c) CA 为网站签署证书**

bin 目录下创建 demoCA 目录，包含：index.txt、serial（内容为01）、空文件夹 newcerts，然后执行：

```
openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key -config ..\conf\openssl.cnf
```


### 4. 将 server.crt，server.key 复制到 apache/conf 文件夹下，重启 apache 

### 5. 访问 https://www.nc.com
