---
title: Server certificate verification failed
date: 2023-09-12 08:49:25
categories:
  - Code
tags: [ssl]
---

在自建服务时原本运行好好的无法返回`server certificate verification failed`.<!-- more --> 


`curl https://xxx.xxx.xxx` 时会告知，`curl: (60) server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none`

但是https网址可以在浏览器内被正常打开，但是通过命令例如git，curl等请求报错，客户端openssl解析证书时存在问题  
以我遇到问题的letsencrypt证书为例，通常时因为请求客户端的openssl，letsencrypt证书签名链种包含了一条`ISRG Root X1`自签名证书链，浏览器会校验该证书链拿到正确的，但是openssl1.0.2不支持此逻辑它依旧拿了旧的过期的`DST Root CA X3`


- 临时解决：  
如果是git的话设置 `git config --global http.sslverify false`  
如果是curl的话请求时加 `-k` 参数跳过ssl证书认证  

- 永久解决：  
在发起请求出错的机器上，升级依赖软件包 `sudo apt update && sudo apt upgrade`  

通过curl请求域名，`curl https://xxx.xxx.xxx` 不报错就说明好了



参考：
- https://letsencrypt.org/docs/dst-root-ca-x3-expiration-september-2021/
- https://community.letsencrypt.org/t/production-chain-changes/150739
- https://www.openssl.org/blog/blog/2021/09/13/LetsEncryptRootCertExpire/
- https://docs.certifytheweb.com/docs/kb/kb-202109-letsencrypt/
- https://stackoverflow.com/questions/21181231/server-certificate-verification-failed-cafile-etc-ssl-certs-ca-certificates-c
