---
layout:     post
title:      "Java Security Providers"
date:       2019-06-02
author:     "YouYou"
header-img: "img/cyber-security.jpg"
tags:
    - java
    - security
---

#java.security
>security.provider.1=sun.security.provider.Sun
security.provider.2=sun.security.rsa.SunRsaSign
security.provider.3=sun.security.ec.SunEC
security.provider.4=com.sun.net.ssl.internal.ssl.Provider
security.provider.5=com.sun.crypto.provider.SunJCE
security.provider.6=sun.security.jgss.SunProvider
security.provider.7=com.sun.security.sasl.Provider
security.provider.8=org.jcp.xml.dsig.internal.dom.XMLDSigRI
security.provider.9=sun.security.smartcardio.SunPCSC
security.provider.10=sun.security.mscapi.SunMSCAPI

#Configure BCFIPS
~~~
Security.insertProviderAt(bcFipsProvider, 1);
~~~
将org.bouncycastle.jcajce.provider.BouncyCastleFipsProvider插入到provider的第一位。jcajce主要用于加密解密。
~~~
Provider sunJSSE = new com.sun.net.ssl.internal.ssl.Provider(bcFipsProvider);
Security.addProvider(sunJSSE);
~~~
SunJSSE是Sun实现的ssl框架。
为SunJSSE配置FIPS 算法实现，此时SunJSSE即为FIPS模式。
