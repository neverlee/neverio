title: Nginx简单https配置
categories: 小笔记
date: 2016-04-05 15:03:09
tags: [nginx,https,ssl]
description: Nginx的简单https配置，用于测试
---

平常测试部署中，免不了要使用https。简单地部署出一个https站点可以这样来。

首先生成证书
```shell
openssl genrsa -out server.pem 2048

openssl req -new -x509 -key server.pem -out cacert.pem -days 1095
# 当出现 Common Name (e.g. server FQDN or YOUR name) []:
# 输入你要部署网站的hostname，例如这里使用example.com
```

之后便可以在nginx中配置使用
```shell
server {
    listen       443;
    server_name  example.com;
    ssl on;
    ssl_certificate keyfile/cacert.pem;
    ssl_certificate_key keyfile/server.pem;
}
```

当然，如果想自己给证书签名，可以这样
```shell
openssl genrsa -out server.pem 2048
openssl req -new -x509 -key server.pem -out cacert.pem -days 1095
# 当出现 Common Name (e.g. server FQDN or YOUR name) []:
# 输入你要部署网站的hostname，例如这里使用example.com

openssl genrsa -des3 -out server.key 1024
openssl req -new -key server.key -out server.csr

openssl rsa -in server.key -out server_nopw.key
# 在加载SSL支持的Nginx并使用上述私钥时除去必须的口令
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

最后可在nginx配置
```shell
server {
    server_name exmaple.com;
    listen 443;
    ssl on;
    ssl_certificate keydir/server.crt;
    ssl_certificate_key keydir/server.key;
}
```
