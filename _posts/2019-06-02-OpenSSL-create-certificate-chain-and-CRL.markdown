---
layout:     post
title:      "OpenSSL Create Certificate"
date:       2019-06-02
author:     "YouYou"
header-img: "img/cyber-security.jpg"
tags:
    - java
    - security
    - OpenSSL
---

参考：https://netnix.org/2014/04/06/self-signed-ca-with-crl-for-java-code-signing/

**1.下载openssl**
**2.配置openssl.cfg 文件**
***
RANDFILE = .rnd

[ req ]
default_md                = sha256
distinguished_name        = req_distinguished_name

[ req_distinguished_name ]
countryName               = Country Name (2 letter code)
countryName_min           = 2
countryName_max           = 2
stateOrProvinceName       = State or Province Name (full name)
localityName              = Locality Name (eg, city)
0.organizationName        = Organization Name (eg, company)
organizationalUnitName    = Organizational Unit Name (eg, section)
commonName                = Common Name (e.g. server FQDN or YOUR name)
commonName_max            = 64
emailAddress              = Email Address
emailAddress_max          = 64

[ v3_req_sign ]
basicConstraints          = CA:FALSE
keyUsage                  = digitalSignature,keyEncipherment
extendedKeyUsage          = serverAuth, clientAuth, codeSigning
crlDistributionPoints     = URI:http://mydomain.com/ca.crl
subjectKeyIdentifier      = hash
authorityKeyIdentifier    = keyid, issuer
subjectAltName 						= @alt_names

[ alt_names ]
DNS.1 = localhost
IP.1 = 10.5.41.95


[ ca ]
default_ca                = ca
private_key               = caca.key
certificate               = caca.crt
database                  = C:\\Users\\huangyou.CORPDOM\\Desktop\\openssl\\ca\\index.txt
serial                    = C:\\Users\\huangyou.CORPDOM\\Desktop\\openssl\\ca\\serial
crl_extensions            = crl_ext
new_certs_dir             = C:\\Users\\huangyou.CORPDOM\\Desktop\\openssl\\cacerts
default_md                = sha256
policy                    = policy_match

[ policy_match ]
countryName               = optional
stateOrProvinceName       = optional
organizationName          = optional
organizationalUnitName    = optional
commonName                = supplied
emailAddress              = optional

[ v3_ca ]
basicConstraints          = critical, CA:TRUE, pathlen:0
keyUsage                  = critical, cRLSign, keyCertSign
subjectKeyIdentifier      = hash
authorityKeyIdentifier    = keyid:always, issuer

[ crl_ext ]
authorityKeyIdentifier    = keyid:always

***

**3.命令行示例**
>1.生成一个enduser证书以及一个csr
>"C:\\Program Files\\OpenSSL-Win64\\bin\\openssl.exe" req -new -newkey rsa:2048 -nodes -keyout enduser4.key -out enduser4.csr -config openssl.cfg

>2.用CA sign csr。如果想要组装成证书链需要把ca的cert和此处生成的cert拼接起来
>"C:\\Program Files\\OpenSSL-Win64\\bin\\openssl.exe" ca -create_serial -days 1095 -in enduser2.csr -out enduser2.crt -notext -extensions v3_req_sign -config op enssl.cfg

***
**4.OpenSSL DB**
OpenSSL DB 就是index.txt文件
![openssl_index.PNG](https://upload-images.jianshu.io/upload_images/8158621-83b3b2b3aabf1a61.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第一列： R 表示revoke, V表示Valid, E 表示Expired(需要update db)
第二列： 过期时间
第三列： 吊销时间戳（R状态下）
第四列： 证书序列号
第五列： subject name

生成crl就是读取这个文件


***
![Self-Signed CA with CRL for Java Code Signing netnix org.png](https://upload-images.jianshu.io/upload_images/8158621-568ae823249ff4b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)











