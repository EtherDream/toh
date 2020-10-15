# ToH (TCP over HTTP)


## 简介

将 TCP 流量封装到 HTTP 中，从而隐藏网站管理服务，减少被攻击的风险。


## 用法

需要 nodejs。

服务端:

    toh -s http-port

客户端:

    toh -c local-port url remote-port


## 演示

服务端:

    ./toh -s 8080

客户端:

    ./toh -c 10022 http://server-ip:8080 22

客户端测试:

    ssh 127.0.0.1 -p 10022


## 应用

本程序不提供认证、加密、日志等功能，所以最好不要运行在公网上，而是通过已有的 Web 服务进行转发，例如 nginx:

```nginx
server {
  server_name                 mysite.com;
  listen                      443 ssl;

  ssl_certificate             mysite.com.cer;
  ssl_certificate_key         mysite.com.key;

  # ...

  location = /ssh-xxxx {
    proxy_pass                http://unix:/tmp/toh.sock;
    proxy_http_version        1.1;
    proxy_set_header          Upgrade $http_upgrade;
    proxy_set_header          Connection upgrade;
    proxy_buffering           off;
    proxy_request_buffering   off;
  }
}
```

服务端:

    rm -f /tmp/toh.sock
    ./toh -s /tmp/toh.sock

客户端:

    ./toh -c 10022 https://mysite.com/ssh-xxxx 22

将 `xxxx` 替换成随机字符串 (例如 `uuidgen`) 以防路径爆破。

将本程序运行为后台服务模式，即可删除服务器防火墙中的 SSH、远程桌面等私有运维服务的端口，对外只公开 Web 端口，减少被攻击的风险。
