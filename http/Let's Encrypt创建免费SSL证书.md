# Let's Encrypt创建免费SSL证书

#### 获取 Certbot 客户端

```
# 下载 Certbot 客户端
$ wget https://dl.eff.org/certbot-auto

# 设为可执行权限
$ chmod a+x certbot-auto
```

#### 申请通配符证书

客户在申请 Let’s Encrypt 证书的时候，需要校验域名的所有权，证明操作者有权利为该域名申请证书，目前支持三种验证方式：

- dns-01：给域名添加一个 DNS TXT 记录。
- http-01：在域名对应的 Web 服务器下放置一个 HTTP well-known URL 资源文件。
- tls-sni-01：在域名对应的 Web 服务器下放置一个 HTTPS well-known URL 资源文件。

使用 Certbot 客户端申请证书方法非常的简单，只需如下一行命令就搞定了。

```
$ ./certbot-auto certonly  -d "*.xxx.com" --manual --preferred-challenges dns-01  --server https://acme-v02.api.letsencrypt.org/directory
```

> 1.申请通配符证书，只能使用 dns-01 的方式。
> 2.`xxx.com` 请根据自己的域名自行更改。

相关参数说明：

```
certonly 表示插件，Certbot 有很多插件。不同的插件都可以申请证书，用户可以根据需要自行选择。
-d 为哪些主机申请证书。如果是通配符，输入 *.xxx.com (根据实际情况替换为你自己的域名)。
--preferred-challenges dns-01，使用 DNS 方式校验域名所有权。
--server，Let's Encrypt ACME v2 版本使用的服务器不同于 v1 版本，需要显示指定。
```

```
输入邮箱
输入A
输入 Y
输入 Y
配置DNS TXT记录
另开一个客户端，dig -t txt ****.***.com @8.8.8.8  验证是否生效，生效后按回车键。 
```

确认生效后，回车继续执行，最后会输出如下内容：

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/xxx.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/xxx.com/privkey.pem
   Your cert will expire on 2018-06-12. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

到了这一步后，恭喜您，证书申请成功。 证书和密钥保存在下列目录：

```
$ tree /etc/letsencrypt/live/xxx.com/
.
├── cert.pem
├── chain.pem
├── fullchain.pem
└── privkey.pem
```

校验证书信息，输入如下命令：

```
$ openssl x509 -in  /etc/letsencrypt/live/xxx.com/cert.pem -noout -text 

# 可以看到证书包含了 SAN 扩展，该扩展的值就是 *.xxx.com
...
Authority Information Access: 
        OCSP - URI:http://ocsp.int-x3.letsencrypt.org
        CA Issuers - URI:http://cert.int-x3.letsencrypt.org/

X509v3 Subject Alternative Name: 
    DNS:*.xxx.com
...
```

### 其它相关

- 证书续期

Let’s encrypt 的免费证书默认有效期为 90 天，到期后如果要续期可以执行：

```
$ certbot-auto renew
```

- 在 Nginx 中 配置 Let’s Encrypt 证书

Nginx 配置文件片断：

```
server {
    server_name xxx.com;
    listen 443 http2 ssl;
    ssl on;
    ssl_certificate /etc/cert/xxx.com/fullchain.pem;
    ssl_certificate_key /etc/cert/xxx.com/privkey.pem;
    ssl_trusted_certificate  /etc/cert/xxx.com/chain.pem;

    location / {
      proxy_pass http://127.0.0.1:6666;
    }
}
```