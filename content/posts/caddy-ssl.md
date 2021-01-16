---
title: "Caddy Ssl"
date: 2020-04-04T18:41:04+08:00
draft: false
tags: ["Caddy","SSL","TLS"]
categories: ["hosting"]
---

Cloudflare的Origin CA是CF自簽的憑證，不通過CF的話會報錯。所以就水一篇文章唄。

本文基於Caddy V1.4

<!--more-->

# 憑證申請

Caddy支持自動申請憑證，包括泛域名（需要dns驗證）

單個域名的申請甚至簡單到：

```visual basic
tls {
	dns cloudflare
}
```

### 通配符

安裝Caddy的時候請添加 `tls.dns.你的提供商`插件

caddy支持很多的dns provider,具體列表請參見官方Github倉庫：https://github.com/caddyserver/dnsproviders

我使用cloudflare管理域名，即使用cf進行dns challenge。需要配置環境變量

```
export CLOUDFLARE_API_KEY="Cloudflare Global Api"  
export CLOUDFLARE_EMAIL="Cloudflare 郵箱"
```

其他的dns服務商請參考下表（數據來源：https://caddyserver.com/v1/docs/automatic-https）

| Provider                       | Name to Use in Caddyfile | Environment Variables to Set                                 |
| ------------------------------ | ------------------------ | ------------------------------------------------------------ |
| Aurora DNS by PCExtreme        | auroradns                | AURORA_USER_ID AURORA_KEY AURORA_ENDPOINT (optional)         |
| Azure DNS                      | azure                    | AZURE_CLIENT_ID AZURE_CLIENT_SECRET AZURE_SUBSCRIPTION_ID AZURE_TENANT_ID |
| Cloudflare                     | cloudflare               | CLOUDFLARE_EMAIL CLOUDFLARE_API_KEY                          |
| CloudXNS                       | cloudxns                 | CLOUDXNS_API_KEY CLOUDXNS_SECRET_KEY                         |
| DigitalOcean                   | digitalocean             | DO_AUTH_TOKEN                                                |
| DNSimple                       | dnsimple                 | DNSIMPLE_EMAIL DNSIMPLE_OAUTH_TOKEN                          |
| DNS Made Easy                  | dnsmadeeasy              | DNSMADEEASY_API_KEY DNSMADEEASY_API_SECRET DNSMADEEASY_SANDBOX (true/false) |
| DNSPod                         | dnspod                   | DNSPOD_API_KEY                                               |
| DynDNS                         | dyn                      | DYN_CUSTOMER_NAME DYN_USER_NAME DYN_PASSWORD                 |
| Gandi                          | gandi / gandiv5          | GANDI_API_KEY / GANDIV5_API_KEY                              |
| GoDaddy                        | godaddy                  | GODADDY_API_KEY GODADDY_API_SECRET                           |
| Google Cloud DNS               | googlecloud              | GCE_PROJECT GCE_DOMAIN GOOGLE_APPLICATION_CREDENTIALS (or GCE_SERVICE_ACCOUNT_FILE) |
| Lightsail by AWS               | lightsail                | AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN (optional) DNS_ZONE (optional) |
| Linode                         | linode                   | LINODE_API_KEY                                               |
| Namecheap                      | namecheap                | NAMECHEAP_API_USER NAMECHEAP_API_KEY                         |
| NS1.                           | ns1                      | NS1_API_KEY                                                  |
| Name.com                       | namedotcom               | NAMECOM_USERNAME NAMECOM_API_TOKEN                           |
| OVH                            | ovh                      | OVH_ENDPOINT OVH_APPLICATION_KEY OVH_APPLICATION_SECRET OVH_CONSUMER_KEY |
| Open Telekom Cloud Managed DNS | otc                      | OTC_DOMAIN_NAME OTC_USER_NAME OTC_PASSWORD OTC_PROJECT_NAME OTC_IDENTITY_ENDPOINT (optional) |
| PowerDNS                       | pdns                     | PDNS_API_URL PDNS_API_KEY                                    |
| Rackspace                      | rackspace                | RACKSPACE_USER RACKSPACE_API_KEY                             |
|                                | rfc2136                  | RFC2136_NAMESERVER RFC2136_TSIG_ALGORITHM RFC2136_TSIG_KEY RFC2136_TSIG_SECRET |
| Route53 by AWS                 | route53                  | AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY                      |
| Vultr                          | vultr                    | VULTR_API_KEY                                                |

**申請出的憑證會自動保存在`~/.caddy/acme/acme-v02.api.letsencrypt.org/sites/wildcard_.你的域名下**`

# 句法

直接從[官網文檔](https://caddyserver.com/v1/docs/tls)抄

需要注意的是Caddy默認最低爲TLS 1.2， 自定義Cipher不支持TLS1.3

```
tls [cert key] {
    ca        uri
    protocols min max
    ciphers   ciphers...
    curves    curves...
    clients   [request|require|verify_if_given] clientcas...
    load      dir
    ask       url
    key_type  type
    dns       provider
    alpn      protos...
    must_staple
    wildcard
}
```

- **ca** specifies the ACME-compatible Certificate Authority endpoint to request certificates from.

- **cert** and **key** are the same as above.

- **protocols** specifies the minimum and maximum protocol  versions to support. See below for valid values. If min and max are the  same, it need only be specified once.

- **ciphers** is a list of space-separated ciphers that will  be supported, overriding the defaults. If you list any, only the ones  you specify will be allowed and preferred in the order given. See below  for valid values. Customizing cipher suites is not allowed with TLS 1.3.

- **curves** is a list of space-separated curves that will be  supported in the given order, overriding the defaults. Valid curves are  listed below.

- clients

   is a list of space-separated client root CAs used  for verification during TLS client authentication. If used, clients will  be asked to present their certificate by their browser, which will be  verified against this list of client certificate authorities. A client  will not be allowed to connect if their certificate was not signed by  one of these root CAs. Note that this setting applies to the entire  listener, not just a single site. You may modify the strictness of  client authentication using one of the keywords before the list of  client CAs: 						

  - **request** merely asks a client to provide a certificate, but will not fail if none is given or if an invalid one is presented.
  - **require** requires a client certificate, but will not verify it.
  - **verify_if_given** will not fail if none is presented, but reject all that do not pass verification.
  - The default, if no flag is set but a CA file found, is to do both: to require client certificates and validate them.

- **load** is a directory from which to load certificates and  keys. The entire directory and its subfolders will be walked in search  of .pem files. Each .pem file must contain the PEM-encoded certificate  (chain) and key blocks, concatenated together.

- **ask** enables [On-Demand TLS](https://caddyserver.com/v1/docs/automatic-https#on-demand).  On-Demand TLS is NOT recommended if the hostname is given in the  Caddyfile and known at configuration-time. The URL will be queried via  GET and should return a 200 status code if the `domain` form  value from the query string is allowed to be given a certificate.  Redirects at this endpoint are not followed. The URL should only be  internally accessible. When using this option, Caddy does not enforce  any extra rate limiting; your endpoint is expected to make wise  decisions instead.

- **key_type** is the type of key to use when generating keys  for certificates (only applies to managed or TLS or self-signed  certificates). Valid values are rsa2048, rsa4096, rsa8192, p256, and  p384. Default is currently p256.

- **dns** is the name of a DNS provider; specifying it enables the [DNS challenge](https://caddyserver.com/v1/docs/automatic-https#dns-challenge) (see that link for details). Note that you need to give credentials via environment variables for it to work.

- **alpn** is a list of space-separated protocols to use for  Application-Layer Protocol Negotiation (ALPN). For HTTPS servers, HTTP  versions can be enabled/disabled with this setting. Default is `h2 http/1.1`.

- **must_staple** enables [Must-Staple](https://blog.mozilla.org/security/2015/11/23/improving-revocation-ocsp-must-staple-and-short-lived-certificates/) for managed certificates. Use with care.

- **wildcard** will obtain and manage a wildcard certificate for this name by replacing the left-most label with `*`,  as long as managed TLS with the DNS challenge is enabled. Any sites  which are configured similarly and have the same resulting wildcard name  will then share the same, single certificate. This will not work with  On-Demand TLS because it uses the SNI value for the certificate name. **Note:** Do not use this feature unless you have many subdomains that would otherwise cause you to hit CA rate limits.