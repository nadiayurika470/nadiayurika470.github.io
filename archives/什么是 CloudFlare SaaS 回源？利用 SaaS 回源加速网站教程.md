早就听说可以利用 SaaS 回源来对网站进行加速，但一直没有尝试过。今天有空便试了一下，特地分享出来供大家参考。

---

## 1. SaaS 回源介绍

### 1.1 什么是 SaaS？

简单来说，SaaS（Software as a Service，软件即服务）是指：当你想要开展某个业务时，不需要自己编写和部署代码，而是通过互联网直接使用现成的软件服务。

例如，使用 Gmail 发邮件、用网盘存储文件、用 Netflix 看视频等，这些都是通过互联网访问某种服务，因此都属于 SaaS。

### 1.2 什么是 SaaS 回源？

对于使用 SaaS 的用户来说，可能希望通过自定义的域名来访问服务。例如，你的公司购买了 Gmail 服务，但为了彰显品牌，不希望使用 @gmail.com 的后缀，而是用公司自己的域名 @xxx.com，这时就需要用到 SaaS 回源。

配置好 SaaS 回源后，访问 xxx.com 的请求会被转发到 gmail.com 进行处理。

### 1.3 为什么能利用 SaaS 回源对网站加速？

对于静态资源（如图片、CSS、JS 等），Cloudflare 在全球的分布式节点可以缓存这些内容。当用户请求这些资源时，访问路径是：

**浏览器 → Cloudflare 节点（国内/最近） → 直接返回缓存内容**

而原来的访问路径是：

**浏览器 → 国外源站**

显然，配置了 SaaS 回源后，不需要请求原站，访问速度会快不少。

---

## 2. 概述

### 需要用到的：

- **【必须】** 你希望加速的域名 `a.com`（不用托管到 Cloudflare）
- **【必须】** 回源域名 `b.com`（必须托管到 Cloudflare）
- **【必须】** 国外信用卡，用于绑定 Cloudflare，推荐使用 [Wildcard | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)
- **【非必须】** DNSPod，用于将**海外线路**和**国内线路**分开解析

### 步骤概述：

1. 将 `b.com` 托管到 Cloudflare，并解析到你的服务器（比如 Github Pages）
2. 配置 Cloudflare SaaS 回源（此功能免费，但需要绑定信用卡），将 `b.com` 作为回退源
3. 在 DNSPod 上，配置 `a.com` 的 DNS，将其指向 Cloudflare

---

## 3. 详细步骤

### 3.1 注册 Cloudflare，并托管 `b.com`

1. 注册并登录 Cloudflare，把 `b.com` 添加进去，并查看 Cloudflare 分配的 NS 服务器。
2. 在你的域名注册商那里，把 DNS 服务器设置为 Cloudflare 提供的 NS 域名。
3. 等待域名状态变为“活动”，表示托管成功。

### 3.2 启用 Cloudflare for SaaS

1. 进入 **SSL → 自定义主机名**，点击启用 Cloudflare SaaS。
2. 绑定外国信用卡。如果没有外国卡，可以使用虚拟卡，推荐 [Wildcard | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)。
3. 在 Wildcard 开卡后，把信息填写到 Cloudflare 即可。

### 3.3 解析回源域名

1. 进入 `b.com` 的管理界面 → **DNS → 记录**。
2. 将 `b.com` 的 A 记录或 AAAA 记录解析到你的真实网站服务器 IP，例如 Github Pages 的服务器。

### 3.4 添加回源

1. 进入 `b.com` 的管理界面 → **SSL/TLS → 自定义主机名**。
2. 添加回退源，回退源地址为刚刚解析的 `b.com`。

### 3.5 添加自定义主机名

1. 添加回源成功后，在同样的界面添加自定义主机名。
2. 主机名为你希望加速的网站域名 `a.com`。
3. 按照 Cloudflare 提示完成域名验证。

### 3.6 配置 `a.com` 的 DNS 解析

1. 在 DNSPod 设置 `a.com` 的域名解析：
   - 境内线路解析到 `shopify.com`。
   - 境外线路解析到 `1.0.0.5`（Cloudflare 服务器在境外的 IP）。

> **提示**：解析到 `shopify.com` 是因为它也使用了 Cloudflare 服务，指向国内最快速的 Cloudflare 节点。

### 3.7 设置 SSL

1. 在 Cloudflare 进入 `b.com` 的管理界面 → **SSL/TLS → 概述**。
2. 将 SSL/TLS 加密模式改为“完全”。

---

## 4. 访问和验证

完成以上配置后，当我们访问 `a.com` 时，请求会先转发给 Cloudflare 的境内节点。境内节点会发现 `a.com` 是 `b.com` 的自定义主机名，直接将缓存的 `b.com` 内容返回给浏览器，从而达到加速网站的目的。

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)