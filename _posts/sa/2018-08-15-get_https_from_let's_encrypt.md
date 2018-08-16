---
layout: post
title: Let’s Encrypt 申请证书
date: 2018-08-15
tags: HTTPS SSL Encrypt
---

### 简述  
Let’s Encrypt 是国外一个公共的免费 SSL 项目，由 Linux 基金会托管。由 Mozilla、思科、Akamai、IdenTrust 和 EFF 等组织发起，目的就是向网站自动签发和管理免费证书。  
Let’s Encrypt 已经得了 IdenTrust 的交叉签名，这意味着其证书现在已经可以被 Mozilla、Google、Microsoft 和 Apple 等主流的浏览器所信任。  

### 环境  
CentOS release 6.5 x86_64

### 申请证书  
Let’s Encrypt 官方推荐使用 `certbot-auto` 申请证书。  
#### 获取脚本  
```shell
[root@Wowking ~]# wget https://dl.eff.org/certbot-auto -P /srv/
[root@Wowking ~]# chmod +x /srv/certbot-auto
```

#### 查看语法  
```shell
[root@Wowking ~]# sh /srv/certbot-auto --help all
Usage: certbot-auto [OPTIONS]
A self-updating wrapper script for the Certbot ACME client. When run, updates
to both this script and certbot will be downloaded and installed. After
ensuring you have the latest versions installed, certbot will be invoked with
all arguments you have provided.

Help for certbot itself cannot be provided until it is installed.

  --debug                                   attempt experimental installation
  -h, --help                                print this help
  -n, --non-interactive, --noninteractive   run without asking for user input
  --no-bootstrap                            do not install OS dependencies
  --no-self-upgrade                         do not download updates
  --os-packages-only                        install OS dependencies and exit
  --install-only                            install certbot, upgrade if needed, and exit
  -v, --verbose                             provide more output
  -q, --quiet                               provide only update/error output;
                                            implies --non-interactive

All arguments are accepted and forwarded to the Certbot client when run.
```

*Tips*  
> run：获取并安装证书到当前的Web服务器  
> certonly：获取或续期证书，但是不安装  
> renew：在证书快过期时，续期之前获取的所有证书  
> -d DOMAINS：一个证书支持多个域名，用逗号分隔  
> --apache：使用 Apache 插件来认证和安装证书  
> --standalone：运行独立的 web server 来验证  
> --nginx：使用 Nginx 插件来认证和安装证书  
> --webroot：如果目标服务器已经有 web server 运行且不能关闭，可以通过往服务器的网站根目录放置文件的方式来验证  
> --manual：通过交互式方式，或 Shell 脚本手动获取证书  


#### 获取证书  
##### 单个域名  
```shell
# 如果 pip 安装有问题，指定源
[root@Wowking ~]# cat ~/.pip/pip.conf
[global]
index-url=http://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
[root@Wowking ~]# /srv/certbot-auto --nginx certonly -m "wowking@xxx.com" -n --agree-tos --domains www.wowking.cc
```

##### 泛域名  
```shell
[root@Wowking ~]# /srv/certbot-auto certonly --manual --agree-tos --no-bootstrap --manual-public-ip-logging-ok --preferred-challenges dns-01 -m "wowking@xxx.com" -d *.wowking.com
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: N
Obtaining a new certificate
Performing the following challenges:
dns-01 challenge for wowking.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.wowking.com with the following value:

1p0NOaP76T1xxxxxxxxxxW6J9jOPryyyyy5s3fapgKY

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/wowking.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/wowking.com/privkey.pem
   Your cert will expire on 2018-11-13. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

*TIPS*  
据说泛域名只能使用 `--manual` 。  
在 `deploy` 之前，先确认 `TXT` 记录已生效。  
```shell
[root@Wowking ~]# dig -t txt _acme-challenge.wowking.com @8.8.8.8

; <<>> DiG 9.8.2rc1-RedHat-9.8.2-0.17.rc1.el6_4.6 <<>> -t txt _acme-challenge.wowking.com @8.8.8.8
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 51841
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;_acme-challenge.wowking.com. IN TXT

;; Query time: 45 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Wed Aug 15 16:50:12 2018
;; MSG SIZE  rcvd: 50
```

### 测试证书安全性  
`https://www.ssllabs.com/ssltest/index.html`  

### 管理证书  
```shell
[root@Wowking ~]# /srv/certbot-auto certificates
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: wowking.com
    Domains: *.wowking.com
    Expiry Date: 2018-11-13 07:45:58+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/wowking.com/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/wowking.com/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

### 注销证书  
```shell
/srv/certbot-auto revoke --cert-path /etc/letsencrypt/live/www.wowking.com/cert.pem 
/srv/certbot-auto delete --cert-name www.wowking.com
```

*TIPS*  
如果没使用 `delete` 注销证书，下次续订活动时会自动 `renew`。  

### 手动续期  
`/srv/certbot-auto renew`  

### 自动续期  
`crontab` 执行  
`0 0,12 * * * python -c 'import random; import time; time.sleep(random.random() * 3600)' && /srv/certbot-auto renew`

*TIPS*  
泛域名暂时不支持自动续期。  


### reference  
https://blog.minirplus.com/11787/  
https://my.oschina.net/longquan/blog/1634420  
https://linuxstory.org/deploy-lets-encrypt-ssl-certificate-with-certbot/  
https://www.hi-linux.com/posts/6968.html  
https://linuxops.org/blog/linux/certbot.html  
