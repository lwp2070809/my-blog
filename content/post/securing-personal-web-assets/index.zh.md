---
title: 在互联网上保护自己的web资产
date: 2026-03-21T14:00:00+08:00
lastmod: 2026-03-21T14:00:00+08:00
draft: false
slug: securing-personal-web-assets
categories: []
tags: ["信息安全", "Cloudflare"]
description: 
---

如今, 任何部署在公网的服务都意味着会受到各种形式的攻击.

对于企业用户, 安全负责人不应该完全承担所有的安全风险, 这不是一个人可以承受的责任, 帮公司节省成本对你来说并没有好处. 直接购买安全公司的服务, 永远是最正确的选择, 各种零信任系统. 企业级VPN, 堡垒机, WAF服务可以在一定程度上保护安全, 属于你的责任就是安全审计.

对于个人用户, 我将以Vaultwarden为例, 将我当前的安全方案, 以及做过的一些研究记录在此.

## Vaultwarden self-hosting安全方案

我的服务器来自于一家可靠的VPS供应商Netcup, 并使用基于Cloudflare的方案进行网络保护. 在此种安全模型中, 你必须足够信任Cloudflare. 安全和便利不可得兼, 而最佳的安全方案永远是本地离线方案.

首先使用docker部署vaultwarden和cloudflared, 这样无需VPS开放端口, 而是使用Cloudflare Tunnel将服务映射到公网. 下面是vaultwarden的部署配置.

**docker-compose.yml**

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    environment:
      - LOG_FILE=/data/vaultwarden.log
      - LOG_LEVEL=warn
      - REAL_IP_HEADER=X-Real-IP
    mem_limit: 256m
    volumes:
      - /opt/docker/vaultwarden/vw-data:/data
    logging:
      driver: local
    restart: always
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    mem_limit: 256m
    command: tunnel --no-autoupdate run --protocol http2 --token my-token
    logging:
      driver: local
    restart: always
```

在Cloudflare One上创建Cloudflare Tunnel以获取token. 在本地cloudflared运行之后, 云上就绑定成功了, 然后服务地址 (类似于反向代理地址的概念) 配置为`http://vaultwarden:80`, 并使用自己在cloudflare里配置的域名创建一个二级域名, 就能应用cloudflare的DNS代理来保护vaultwarden服务, 包括网页和API端点.

但这还远远称不上安全. 自建的vaultwarden理应只能让自己访问, 即使配置了两步验证和fail2ban, 暴露在公网上依然可能受到0day的攻击. 接下来将基于代理服务器和cloudflare Security rules构建进一步的保护. 这里的前提条件: 你的设备上必须常驻一个代理或VPN. 这样可以基于简单的IP白名单策略来保护vaultwarden. 下面的方案基于我的实际情况:

我在自己的几个VPS上, 使用某个工具搭建了基于TCP的代理服务, 并在路由策略中, 配置了vaultwarden的域名为服务器直出, 示例如下:

```json
{
  "type": "field",
  "domain": [
    "domain:example.com"
  ],
  "outboundTag": "direct"
}
```

这样只要我连接了代理, 就会永远使用VPS的固定IP地址对vaultwarden的域名进行访问, 因此也可以很简单的在cloudflare编写规则

```
(not ip.src in {your-vps-ip-1 your-vps-ip-2} and http.host eq "vaultwarden.example.com")
```

并将措施设为阻止.

这样, 只有你自己的设备, 开启了你自己搭建的代理服务器, 并让vaultwarden, 无论是网页, 浏览器扩展还是APP, 都通过代理, 这样才能成功访问vaultwarden, 否则其他方式访问都会直接被cloudflare给block.

## 先前的方案

最初我没有使用任何其他商业服务, 仅仅是使用Caddy的Auto HTTPS和vaultwarden自带的TOTP.

接下来我开始利用cloudflare进行安全保护. 最开始, 只使用了cloudflare DNS代理, 这会使得Caddy默认Auto HTTPS的HTTP challenge模式失效, 因此我配置了DNS challenge, 这增加了一层复杂度. 但是这就保证了即使是cloudflare访问我的vaultwarden服务也经过https, 实际上在这一层拥有比Cloudflare Tunnel更高的安全性, 因为Cloudflare Tunnel是通过http来访问内部的vaultwarden. 因此如果你不够信任cloudflare, 那其实更应该使用这种方式. 你还需要全局配置trusted_proxies, 确保caddy只能被cloudflare的IP地址进行访问. Caddy必须使用第三方模块构建来实现DNS challenge和trusted_proxies cloudflare.

```
{
  servers {
    trusted_proxies cloudflare {
      interval 12h
      timeout 15s
    }
  }
}
```

然后就是配置fail2ban, 这里需要在caddy的反向代理中配置, 将cloudflare里携带的用户真实IP传递给vaultwarden

```
reverse_proxy localhost:50000 {
    header_up X-Real-IP {http.request.header.Cf-Connecting-Ip}
}   
```

fail2ban的具体细节不做描述, 因为配置非常的复杂而且易出错. 具体的原理就是fail2ban解析vaultwarden日志中记录的, 短时间内多次输错密码或TOTP的IP地址, 并调用cloudflare API, 在Security rules中ban掉对应的IP地址.

## 小结

只要你能明白每种方案的安全边界, 那么综合自己的实际情况, 比如便利性要求和数据安全级别, 完全可以组合各种安全方案. 但我仍然建议HTTPS和TOTP是绝对不可或缺的.

如果你使用的是国内的服务器, 或者本地NAS, 首先使用wireguard等VPN接入到云上VPC或者本地局域网, 可以非常简单的得到访问安全性. 只要不将端口暴露在公网上, 那总是安全的. 唯一需要暴露的是VPN端口, 但绝大多数流行的VPN从设计上就是一个非常安全的模型.