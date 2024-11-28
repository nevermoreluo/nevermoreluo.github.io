---
title: 个人vps获取免费的https证书的几个方法
date: 2024-11-28 14:36:41
categories:
  - Tools
tags: [linux]
---


首先，如果你连域名都没有，也别跑，方法2可以借用一些赛博菩萨让你照样拥有一个https域名。
简单归类一下几个免费获取途径
- 最简单最方便的就是[let's encrypt](https://letsencrypt.org/)
- 通过[traefik.me](https://traefik.me/) 获得自己ip的域名，并且支持https
- 通过赛博菩萨[cloudflare](https://cloudflare.com) pages添加反代（注意流量少免费，但是自己的业务一般不会超
下面来详细记录下各自都该如何处理吧<!--more-->

### let's encrypt
没啥好说的下载certbot，跟着配置文档一把梭，国外的vps一般都没啥问题，市面上教程也很多，国内的vps各钟备案审核各自有各自的问题，自行解决或者改其他方案吧


### traefik.me
实际是通过域名的特性实现的，原理就是访问10-0-0-1.traefik.me实际解析得到的是10.0.0.1，这个时候将10-0-0-1替换成你的原站ip就好了。  
等等那么证书呢？直接在[官网链接](https://traefik.me/#:~:text=Thanks%20to%20Let%27s%20encrypt%2C%20a%20wildcard%20certificate%20is%20available%20for%20*.traefik.me.%0AJust%20grab%20the%20files%20here)下载,  
需要注意以下两点
- 1. 因为它也是Let's encrypt生成的，所以3个月（目前是3个月，我不确定之后是多久）会过期，写个crontab定时更新下就搞定了
- 2. 只支持一级域名的泛域名，所以虽然dns可以通过1.2.4.8.traefik.me解析到1.2.4.8，但是证书会提示不合法，所以需要使用1-2-4-8.traefik.me这种格式


### Cloudflare pages
操作上比较复杂，有些步骤可能会一笔带过，以下用cf简称cloudflare
- 在cf上创建一个dns解析到源服务器，假设想解析的是tv.aaa.com,那么就创建一个tv-cf-origin.aaa.com,以下提到该域名时请自动替换到你拥有的域名
- 配置源站ssl，从cf的SSL/TLS->源服务器,生成一个证书，注意这个私钥证书需要自己保存，只有第一次生成的时候会展示，之后在cf上都再也找不到了，然后将证书配置到源站上，这个源站上域名写tv-cf-origin.aaa.com（这个证书仅仅能用于cf内部，外部由于没有授信的根证书所以还是会失败的，但是这已经足够了）
以下是nginx配置示例： 
nginx-tv-cf-origin-aaa-com.conf
```
server {
    listen 443 ssl;
    server_name tv-cf-origin.aaa.com;
    ssl_certificate /etc/nginx/ssl/tv-cf-origin.aaa.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/tv-cf-origin.aaa.com/privkey.pem;

    location / {
       alias /data/tv/gao/;
       try_files $uri $uri/ =404;
    }
}
```

- 创建github仓库，在github上创建一个私有仓库，新建文件_worker.js
```
export default {
  async fetch(request, env, ctx) {
    let url = new URL(request.url);
    if (url.pathname.startsWith('/')) {
      url.hostname = 'tv-cf-origin.aaa.com'
      let new_request = new Request(url, request);
      return fetch(new_request);
    }
    return env.ASSETS.fetch(request);
  },
};
```
- 创建pages，在cf上打开Workers 和 Pages, 点击Pages，从github上导入刚才创建的私有仓库，等待部署完成
- 修改pages自定义域名，点击刚才创建的pages->自定义域->设置自定义域， 这里才是填的你真正想用的域名，添加tv.aaa.com，cf会自动完成域名解析
- 访问https://tv.aaa.com

