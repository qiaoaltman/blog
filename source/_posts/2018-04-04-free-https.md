---
layout: post
title: "Let's Encrypt，免费好用的 HTTPS 证书"
date: 2018-04-04 19:37:27 +0800
comments: true
categories: HTTPS
---
很早之前我就在关注 [Let's Encrypt](https://letsencrypt.org) 这个免费、自动化、开放的证书签发服务。它由 ISRG（Internet Security Research Group，互联网安全研究小组）提供服务，而 ISRG 是来自于美国加利福尼亚州的一个公益组织。Let's Encrypt 得到了 Mozilla、Cisco、Akamai、Electronic Frontier Foundation 和 Chrome 等众多公司和机构的支持，发展十分迅猛。

申请 Let's Encrypt 证书不但免费，还非常简单，虽然每次只有 90 天的有效期，但可以通过脚本定期更新，配好之后一劳永逸。经过一段时间的观望，我也正式启用 Let's Encrypt 证书了，本文记录本站申请过程和遇到的问题。<!--more-->

我没有使用 Let's Encrypt 官网提供的工具来申请证书，而是用了 [acme-tiny](https://github.com/diafygi/acme-tiny) 这个更为小巧的开源工具。以下内容基本按照 acme-tiny 的说明文档写的，省略了一些我不需要的步骤。

ACME 全称是 Automated Certificate Management Environment，直译过来是自动化证书管理环境的意思，Let's Encrypt 的证书签发过程使用的就是 ACME 协议。有关 ACME 协议的更多资料可以在[这个仓库](https://github.com/ietf-wg-acme/acme/)找到。

### 创建帐号

首先创建一个目录，例如 `ssl`，用来存放各种临时文件和最后的证书文件。进入这个目录，创建一个 RSA 私钥用于 Let's Encrypt 识别你的身份：

```bash
openssl genrsa 4096 > account.key
```

### 创建 CSR 文件

接着就可以生成 CSR（Certificate Signing Request，证书签名请求）文件了。在这之前，还需要创建域名私钥（一定不要使用上面的账户私钥），根据证书不同类型，域名私钥也可以选择 RSA 和 ECC 两种不同类型。以下两种方式请根据实际情况二选一。

1）创建 RSA 私钥（兼容性好）：

```bash
openssl genrsa 4096 > domain.key
```

2）创建 ECC 私钥（部分老旧操作系统、浏览器不支持。优点是证书体积小）：

```bash
#secp256r1
openssl ecparam -genkey -name secp256r1 | openssl ec -out domain.key

#secp384r1
openssl ecparam -genkey -name secp384r1 | openssl ec -out domain.key
```

有关 ECC 证书的更多介绍，请[点击这里](https://imququ.com/post/optimize-tls-handshake.html#toc-2-1)。

有了私钥文件，就可以生成 CSR 文件了。在 CSR 中推荐至少把域名带 `www` 和不带 `www` 的两种情况都加进去，其它子域可以根据需要添加（目前一张证书最多可以包含 100 个域名）：

```bash
openssl req -new -sha256 -key domain.key -subj "/" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:yoursite.com,DNS:www.yoursite.com")) > domain.csr
```

执行这一步时，如果提示找不到 `/etc/ssl/openssl.cnf` 文件，请看看 `/usr/local/openssl/ssl/openssl.cnf` 是否存在。如果还是不行，也可以使用交互方式创建 CSR（需要注意 Common Name 必须为你的域名）：

```bash
openssl req -new -sha256 -key domain.key -out domain.csr
```

### 配置验证服务

我们知道，CA 在签发 DV（Domain Validation）证书时，需要验证域名所有权。传统 CA 的验证方式一般是往 `admin@yoursite.com` 发验证邮件，而 Let's Encrypt 是在你的服务器上生成一个随机验证文件，再通过创建 CSR 时指定的域名访问，如果可以访问则表明你对这个域名有控制权。

首先创建用于存放验证文件的目录，例如：

```bash
mkdir ~/www/challenges/
```

然后配置一个 HTTP 服务，以 Nginx 为例：

```nginx
server {
    server_name www.yoursite.com yoursite.com;

    location ^~ /.well-known/acme-challenge/ {
        alias /home/xxx/www/challenges/;
        try_files $uri =404;
    }

    location / {
        rewrite ^/(.*)$ https://yoursite.com/$1 permanent;
    }
}
```

以上配置优先查找 `~/www/challenges/` 目录下的文件，如果找不到就重定向到 HTTPS 地址。这个验证服务以后更新证书还要用到，建议一直保留。

### 获取网站证书

先把 acme-tiny 脚本保存到之前的 `ssl` 目录：

```bash
wget https://raw.githubusercontent.com/diafygi/acme-tiny/master/acme_tiny.py
```

指定账户私钥、CSR 以及验证目录，执行脚本：

```bash
python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir ~/www/challenges/ > ./signed.crt
```

如果一切正常，当前目录下就会生成一个 `signed.crt`，这就是申请好的证书文件。

如果你把域名 DNS 解析放在国内，这一步很可能会遇到类似这样的错误：

```
ValueError: Wrote file to /home/xxx/www/challenges/oJbvpIhkwkBGBAQUklWJXyC8VbWAdQqlgpwUJkgC1Vg, but couldn't download http://www.yoursite.com/.well-known/acme-challenge/oJbvpIhkwkBGBAQUklWJXyC8VbWAdQqlgpwUJkgC1Vg
```

这是因为你的域名很可能在国外无法解析，可以找台国外 VPS 验证下。我的域名最近从 DNSPod 换到了阿里云解析，最后又换到了 CloudXNS，就是因为最近前两家在国外都很不稳定。如果你也遇到了类似情况，可以暂时使用国外的 DNS 解析服务商，例如 [dns.he.net](https://dns.he.net/)。如果还是搞不定，也可以试试「[Neilpang/le](https://github.com/Neilpang/le)」这个工具的 DNS Mode。

搞定网站证书后，还要下载 Let's Encrypt 的中间证书。我在之前的文章中讲过，配置 HTTPS 证书时既不要漏掉中间证书，也不要包含根证书。在 Nginx 配置中，需要把中间证书和网站证书合在一起：

```bash
wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem
cat signed.crt intermediate.pem > chained.pem
```

为了后续能顺利启用 [OCSP Stapling](https://imququ.com/post/why-can-not-turn-on-ocsp-stapling.html#toc-2)，我们再把根证书和中间证书合在一起：

```bash
wget -O - https://letsencrypt.org/certs/isrgrootx1.pem > root.pem
cat intermediate.pem root.pem > full_chained.pem
```

最终，修改 Nginx 中有关证书的配置并 reload 服务即可：

```nginx
ssl_certificate     ~/www/ssl/chained.pem;
ssl_certificate_key ~/www/ssl/domain.key;
```

Nginx 中与 HTTPS 有关的配置项很多，这里不一一列举了。如有需要，[请参考本站配置](https://imququ.com/post/my-nginx-conf.html)。

### 配置自动更新

Let's Encrypt 签发的证书只有 90 天有效期，推荐使用脚本定期更新。例如我就创建了一个 `renew_cert.sh` 并通过 `chmod a+x renew_cert.sh` 赋予执行权限。文件内容如下：

```bash
#!/bin/bash

cd /home/xxx/www/ssl/
python acme_tiny.py --account-key account.key --csr domain.csr --acme-dir /home/xxx/www/challenges/ > signed.crt || exit
wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem
cat signed.crt intermediate.pem > chained.pem
service nginx reload
```

crontab 中使用绝对路径比较保险，`crontab -e` 加入以下内容：

```bash
0 0 1 * * /home/xxx/shell/renew_cert.sh >/dev/null 2>&1
```

这样以后证书每个月都会自动更新，一劳永逸。实际上，Let's Encrypt 官方将证书有效期定为 90 天一方面是为了更安全，更重要的是鼓励用户采用自动化部署方案。

### 几个问题

Let's Encrypt 证书的兼容性，所有操作系统、浏览器默认是否都能识别是大家最关心的问题。实际上，由于 Let's Encrypt 与 IdenTrust 的 DST Root CA 做了交叉认证，兼容性还是不错的，目前我只是发现在 Android 2 和 Windows XP 下有问题（Firefox 的证书那一套是自己实现的，不依赖于系统，XP 下只有 Firefox 信任 Let's Encrypt 证书），其它环境都正常。

<img alt="letsencrypt intermediate cert on winxp" src="https://st.imququ.com/static/uploads/2015/12/letsencrypt-intermediate-cert-on-winxp.png" width="413" itemprop="image" />（Windows XP 不信任 Let's Encrypt 的中间证书）

更新：根据 Let's Encrypt 官方说明，Windows XP 下的问题很快就会解决：

> A bug in Windows XP causes parsing of our current cross-signature from IdenTrust to fail. We will be correcting this by getting new cross-signatures from IdenTrust which work on Windows XP.<br />
> 注：已于 2016 年 3 月 26 日解决。

另外一个问题有关 ECC 证书，官网表示计划将在 2016 年提供对 ECC 证书的支持：

> Right now all of our root and intermediate keys use RSA. We're planning to generate ECC keys and make an ECC option available to subscribers in 2016. [via](https://community.letsencrypt.org/t/elliptic-curve-cryptography-ecc-support/34)<br />
> 注：Let's Encrypt 已于 2016 年 2 月 11 日开始支持签发 ECC 证书。

Let's Encrypt 官方的新特性预告可以在[这个页面查看](https://letsencrypt.org/upcoming-features/)。

我个人建议：对于个人用户来说，如果非常在意证书兼容性，可以购买 RapidSSL Standard 或者 Comodo Positive SSL 这两种证书。其中 RapidSSL 证书一共才三级，比较小；Comodo Positive 有四级，但可以申请 ECC 证书；二者都有着不错的兼容性，也非常廉价（一年不到 10$）。当然，如果不用考虑 Windows XP 用户，那么强烈推荐 Let's Encrypt！

更新：Let's Encrypt 已经支持 Windows XP 和签发 ECC 证书，对于个人用户来说，目前 Let's Encrypt 无疑是最好的选择。

本文先写到这里，如果你在申请 Let's Encrypt 证书的过程中遇到问题，可以给我留言，也欢迎交流各种心得！

原文链接：[https://imququ.com/post/letsencrypt-certificate.html](https://imququ.com/post/letsencrypt-certificate.html)，[前往原文评论 »](https://imququ.com/post/letsencrypt-certificate.html#comments)
