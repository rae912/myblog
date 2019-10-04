
---
layout: post
title: "Java网络访问绕过HTTPS的证书"
subtitle: "Java without SSL when HTTPS"
author: "qingshan"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---


# Java网络访问绕过

今天在内网通过Java访问一个业务接口，测试环境好好的，发布到正式环境就死活不通，一直在报这个错误：Exception in thread "main" javax.net.ssl.SSLHandshakeException:。

经过询问老同事，答复这个问题许多人都遇到过，原因是公司自己签发了证书，但是后端配置证书不正确。解决办法也很简单，就是绕过。因为是内网应用，所以没有安全风险。

Java是通过HttpURLConnection来进行网络访问的，只需要在网络访问的类的开头，直接添加如下的代码即可忽略证书校验了。

    static 
      {
        try
        {
          trustAllHttpsCertificates();
          HttpsURLConnection.setDefaultHostnameVerifier
          (
            new HostnameVerifier() 
            {
              public boolean verify(String urlHostName, SSLSession session)
              {
                return true;
              }
            }
          );
        } catch (Exception e)  {}
      }
    

    private static void trustAllHttpsCertificates()
                throws NoSuchAlgorithmException, KeyManagementException
        {
            TrustManager[] trustAllCerts = new TrustManager[1];
            trustAllCerts[0] = new TrustAllManager();
            SSLContext sc = SSLContext.getInstance("SSL");
            sc.init(null, trustAllCerts, null);
            HttpsURLConnection.setDefaultSSLSocketFactory(
                    sc.getSocketFactory());
        }
    
        private static class TrustAllManager
                implements X509TrustManager
        {
            public X509Certificate[] getAcceptedIssuers()
            {
                return null;
            }
            public void checkServerTrusted(X509Certificate[] certs,
                                           String authType)
                    throws CertificateException
            {
            }
            public void checkClientTrusted(X509Certificate[] certs,
                                           String authType)
                    throws CertificateException
            {
            }
        }