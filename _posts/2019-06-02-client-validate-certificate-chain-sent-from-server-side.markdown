---
layout:     post
title:      "Client Validate Certificate"
date:       2019-06-02
author:     "YouYou"
header-img: "img/cyber-security.jpg"
tags:
    - java
    - security
---

java提供了工具用来支持校验certificate，包括证书的属性校验以及revocation校验。

##Revocation校验
证书的revocation 校验主要有三种方式：
  1.自己提供CRLfiles
  2.通过证书内的CRLDP属性下载CRL files 然后做检查
  3.将待检查的证书发给证书内的OCSP地址，然后检查返回结果

~~~Security.setProperty("ocsp.enable", "true");
            System.setProperty("com.sun.security.enableCRLDP","true");
            List<X509Certificate> certList = Arrays.asList(x509Certificates);
            Set<TrustAnchor> trustAnchors = new HashSet<TrustAnchor>();
            for (X509Certificate trustedRootCert : getAcceptedIssuers()) {
                trustAnchors.add(new TrustAnchor(trustedRootCert, null));
            }
            PKIXBuilderParameters params = new PKIXBuilderParameters(trustAnchors, new X509CertSelector());
            params.addCertStore(CertStore.getInstance("Collection", new CollectionCertStoreParameters(SSLHelper.loadCRL())));
            CertificateFactory cf = CertificateFactory.getInstance("X.509");
            CertPath certPath = cf.generateCertPath(certList);
            CertPathValidator certPathValidator = CertPathValidator.getInstance(CertPathValidator.getDefaultType());
            PKIXRevocationChecker revocationChecker = (PKIXRevocationChecker) certPathValidator.getRevocationChecker();
            revocationChecker.setOptions(EnumSet.of(PKIXRevocationChecker.Option.SOFT_FAIL));
            params.addCertPathChecker(revocationChecker);
            certPathValidator.validate(certPath, params);
~~~
##SunJSSE的实现:
 validate方法会调用一系列的checker做证书检查，包括RevocationChecker，Basic Checker

##BouncyCastleFIPS实现：
目前所有的校验都写到了org.bouncycastle.jcajce.provider.PKIXCertPathValidatorSpi#engineValidate 这一个方法里，暂时没有实现java.security.cert.CertPathValidatorSpi#engineGetRevocationChecker方法。

##FIPS 模式下client端如何对server的证书进行校验
背景：
SunJSSE在FIPS 模式下不允许对x509trustmanager进行扩展。
解决方案：
在TrustManagerFactory初始化时传入PKIXBuilderParameters 。
对于需要检查的证书参数，将其传入X509CertSelector中。
X509CertSelector 只会作用于user certificate.(BCFIPS源码)
~~~
        X509CertSelector x509CertSelector = new X509CertSelector();
        x509CertSelector.setKeyUsage(new boolean[]{true,false,true});
        x509CertSelector.setExtendedKeyUsage(Sets.newHashSet(EXTENDED_KEY_SERVER_AUTHENTICATION,EXTENDED_KEY_CLIENT_AUTHENTICATION));
        PKIXBuilderParameters params = new PKIXBuilderParameters(ts, x509CertSelector);
        params.addCertStore(CertStore.getInstance("Collection", new CollectionCertStoreParameters(SSLHelper.loadCRL())));
        params.setRevocationEnabled(true);
        tmf.init(new CertPathTrustManagerParameters(params));
~~~







