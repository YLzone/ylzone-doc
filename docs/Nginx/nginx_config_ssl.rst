====================
[配置]-HTTPS证书配置
====================

1 背景介绍
----------------

1.1 部署说明::
    
    软件版本: nginx-1.10.1
    所需目录: /data/nginx/data/cert

1.2 所需文件::

    证书文件: www.ylzone.pem
    私钥文件: www.ylzone.key 

2 解决依赖
----------

2.1 创建所需目录::

    $ mkdir /data/nginx/data/cere
    $ chown -R root:root /data/nginx/data/cere
    $ chmod 400 /data/nginx/data/cert/www.ylzone.pem
    $ chmod 400 /data/nginx/data/cert/www.ylzone.key

3 修改配置
----------

3.1 修改配置文件:

.. code-block:: bash
    
    $ vim /etc/nginx/conf/conf.d/server_ylzone.conf
    # 添加如下内容:
    server {
        listen 443;
        server_name www.ylzone.com;

        root html;
        index index.html;

        ssl on;
        ssl_certificate   /data/nginx/data/cert/www.ylzone.pem;
        ssl_certificate_key  /data/nginx/data/cert/www.ylzone.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_prefer_server_ciphers on;

        location / {
            root html;
            index index.html;
        }
    }

3.2 扩展配置:

.. code-block:: bash

    # 要让https和http并存，不能在配置文件中使用ssl on，配置listen 443 ssl
    $ vim /etc/nginx/conf/conf.d/server_ylzone.conf
    # 添加如下内容:
    server {
        listen 80;
        listen 443 ssl;
        server_name www.ylzone.com;

        root html;
        index index.html;

        ssl_certificate   /data/nginx/data/cert/www.ylzone.pem;
        ssl_certificate_key  /data/nginx/data/cert/www.ylzone.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_prefer_server_ciphers on;
    }

3.3 主要配置说明:

``ssl_certificate``:
    证书文件存放位置
    
``ssl_certificate_key``:
    私钥文件存放位置

``ssl_session_timeout``
    ssl会话缓存存活时间(默认5分钟)

``ssl_protocols``
    指定ssl协议

``ssl_ciphers``
    指定ssl算法

``ssl_prefer_server_ciphers``


4 应用配置
----------

4.1 加载配置::

    $ nginx -s reload

4.2 验证配置:

.. code-block:: bash

    # openssl s_client -connect xwsld.jiangrongxin.com:443
    CONNECTED(00000003)
    depth=2 C = US, O = "VeriSign, Inc.", OU = VeriSign Trust Network, OU = "(c) 2006 VeriSign, Inc. - For authorized use only", CN = VeriSign Class 3 Public Primary Certification Authority - G5
    verify return:1
    depth=1 C = US, O = Symantec Corporation, OU = Symantec Trust Network, OU = Domain Validated SSL, CN = Symantec Basic DV SSL CA - G1
    verify return:1
    depth=0 CN = hbh5.jiangrongxin.com
    verify return:1
    ---
    Certificate chain
     0 s:/CN=hbh5.jiangrongxin.com
       i:/C=US/O=Symantec Corporation/OU=Symantec Trust Network/OU=Domain Validated SSL/CN=Symantec Basic DV SSL CA - G1
     1 s:/C=US/O=Symantec Corporation/OU=Symantec Trust Network/OU=Domain Validated SSL/CN=Symantec Basic DV SSL CA - G1
       i:/C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=(c) 2006 VeriSign, Inc. - For authorized use only/CN=VeriSign Class 3 Public Primary Certification Authority - G5
    ---
    Server certificate
    -----BEGIN CERTIFICATE-----
    MIIFhjCCBG6gAwIBAgIQS3op4PZgv1aTSmHwhoAB6jANBgkqhkiG9w0BAQsFADCB
    lDELMAkGA1UEBhMCVVMxHTAbBgNVBAoTFFN5bWFudGVjIENvcnBvcmF0aW9uMR8w
    HQYDVQQLExZTeW1hbnRlYyBUcnVzdCBOZXR3b3JrMR0wGwYDVQQLExREb21haW4g
    VmFsaWRhdGVkIFNTTDEmMCQGA1UEAxMdU3ltYW50ZWMgQmFzaWMgRFYgU1NMIENB
    IC0gRzEwHhcNMTcwMjEwMDAwMDAwWhcNMTgwMjEwMjM1OTU5WjAgMR4wHAYDVQQD
    DBVoYmg1LmppYW5ncm9uZ3hpbi5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAw
    ggEKAoIBAQCWR2GAAo3mxjf9LXsiXIUnZRhTrjPpnpfcZ7jp3Zix4NoyMnkSK4qk
    rMl6jihVJU5izD7I0cgi2OtXuXyvxeeNANu7EQDnC9OaOT3eDEBDXP0qmBc0Mvu3
    ldMgQ20nZ+c+C0op+enBzK8K31NnhTZG5O2kcG/2yqTZ/nHjLYk7VokwGEr7vl8t
    diq4XRFxLw2wSlfwsvM3CwVzETUxvQHJ55bLRCcmnAb4q09NvyDf8NLbvcFkN20Q
    fuz/JwQQgRkkUipk05xLKZ3BXPP0iB9JFdj02MeuD9EInvLPqLpBRHPT5+BgMT9q
    TEBCd9IrkjWT48w74FlB6YAAAdFon2fNAgMBAAGjggJFMIICQTAgBgNVHREEGTAX
    ghVoYmg1LmppYW5ncm9uZ3hpbi5jb20wCQYDVR0TBAIwADBhBgNVHSAEWjBYMFYG
    BmeBDAECATBMMCMGCCsGAQUFBwIBFhdodHRwczovL2Quc3ltY2IuY29tL2NwczAl
    BggrBgEFBQcCAjAZDBdodHRwczovL2Quc3ltY2IuY29tL3JwYTAfBgNVHSMEGDAW
    gBRcYZ6wdkGpaqpDC+HHbjApbrHNNjAOBgNVHQ8BAf8EBAMCBaAwHQYDVR0lBBYw
    FAYIKwYBBQUHAwEGCCsGAQUFBwMCMFcGCCsGAQUFBwEBBEswSTAfBggrBgEFBQcw
    AYYTaHR0cDovL2hjLnN5bWNkLmNvbTAmBggrBgEFBQcwAoYaaHR0cDovL2hjLnN5
    bWNiLmNvbS9oYy5jcnQwggEEBgorBgEEAdZ5AgQCBIH1BIHyAPAAdgDd6x0reg1P
    piCLga2BaHB+Lo6dAdVciI09EcTNtuy+zAAAAVonVF7RAAAEAwBHMEUCIQD7GayM
    YmL+yPY77Hr5t7+kfAg99xBgLV4azK4lhs+/SAIgIzdGHdQdiSxvvFaPXtTfX6NS
    oJJdqSGf45j0Aw4YLUMAdgCkuQmQtBhYFIe7E6LMZ3AKPDWYBPkb37jjd80OyA3c
    EAAAAVonVF8LAAAEAwBHMEUCIQCBlViOgKpWm8qy3dRnw6HudqyS+GcDZyBvTQe3
    aiucSgIgU6heqbZ+bA5ZTytR1zN1Vwql4q3TunGkR2ov943G/tQwDQYJKoZIhvcN
    AQELBQADggEBAIkmhKCpuoAa4+hWhpDiTaGyHJPa66NV7AJMmAnPhfNmrn71ZTyw
    CNYEKXfaD2gTnagiN9S1dxymDg82mDTgEKmGuMltjDxICyZd9nDG3cWKjncVZaVh
    hrbEu/Bg6Y23iOwpPglrV/KFzbimfmGSe9lwRkiVzA01l7TLl+glhXmAJf08nylm
    I2Txd1YCXDuuoOah/c4BcEOpORU67lql6jo+oIGydq+r3KsXt57yBVYKfeP3NO9B
    47Ak5Z3euXyIq1EvITcMvD0xg2UQvlaHMLK/zXLeP8lQ7wsHPv1VMawYIkXwSEbh
    evJ2Z8xY2AZw2+t8H10L3JdPNfxdOmh4Ceg=
    -----END CERTIFICATE-----
    subject=/CN=hbh5.jiangrongxin.com
    issuer=/C=US/O=Symantec Corporation/OU=Symantec Trust Network/OU=Domain Validated SSL/CN=Symantec Basic DV SSL CA - G1
    ---
    No client certificate CA names sent
    Server Temp Key: ECDH, prime256v1, 256 bits
    ---
    SSL handshake has read 3478 bytes and written 373 bytes
    ---
    New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES256-GCM-SHA384
    Server public key is 2048 bit
    Secure Renegotiation IS supported
    Compression: NONE
    Expansion: NONE
    SSL-Session:
        Protocol  : TLSv1.2
        Cipher    : ECDHE-RSA-AES256-GCM-SHA384
        Session-ID: DD3B3559E6EE88789E9346120001BB63414F21565A7735E46EABD614598E8B2C
        Session-ID-ctx: 
        Master-Key: 0EF79F01B00EC500ECDC05A34EF02A729F4DA8922E455B144FEEF4755569C1C2E3285C53E4F05E2BA3EDEF025DCDF019
        Key-Arg   : None
        Krb5 Principal: None
        PSK identity: None
        PSK identity hint: None
        TLS session ticket lifetime hint: 300 (seconds)
        TLS session ticket:
        0000 - a4 c4 35 f6 1d fb 03 fa-f7 8d 4b 78 ca c3 91 2a   ..5.......Kx...*
        0010 - 6d 0d 15 bf 12 07 5b ff-f8 4b c0 67 13 a3 4d bb   m.....[..K.g..M.
        0020 - b4 8c 2e a3 84 08 b6 bc-64 f6 80 d5 f9 f1 a0 0f   ........d.......
        0030 - 63 70 f3 04 ad 77 e4 c1-96 09 d7 cf 50 58 c0 a1   cp...w......PX..
        0040 - dc 46 ac 94 22 86 c3 a5-1c 00 a3 b4 9c d6 50 3c   .F..".........P<
        0050 - 4e 11 c9 f8 c9 52 c6 30-b6 d4 29 9b da 6e f9 0e   N....R.0..)..n..
        0060 - 1f 1b 33 99 c1 df a0 c1-34 14 19 89 68 96 16 ad   ..3.....4...h...
        0070 - a0 31 b1 5b 2e 82 3d 9d-97 c1 c4 76 46 4f eb 30   .1.[..=....vFO.0
        0080 - 14 45 17 fb bb 90 4c 15-c1 c7 6c a2 f9 6b cd e1   .E....L...l..k..
        0090 - d3 52 62 fb fc 62 0d c0-23 1b 06 e6 a5 0e 3d a2   .Rb..b..#.....=.
        00a0 - 63 d9 c7 7c 3d 6a 3e cf-cc b1 ea a1 39 bb 4c 56   c..|=j>.....9.LV
    
        Start Time: 1491871188
        Timeout   : 300 (sec)
        Verify return code: 0 (ok)
    ---
    closed

5 详细说明
----------

5.1 基本介绍

配置HTTPS主机，必须在server配置块中打开SSL协议，还需要指定服务器端证书和密钥文件的位置::

    server {
        listen              443;
        server_name         www.example.com;
        ssl                 on;
        ssl_certificate     www.example.com.crt;
        ssl_certificate_key www.example.com.key;
        ssl_protocols       SSLv3 TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        ...
    }


服务器证书是公开的，会被传送到每一个连接到服务器的客户端。而私钥不是公开的，需要存放在访问受限的文件中，当然，nginx主进程必须有读取密钥的权限。私钥和证书可以存放在同一个文件中::

    ssl_certificate     www.example.com.cert;
    ssl_certificate_key www.example.com.cert;

这种情况下，证书文件同样得设置访问限制。当然，虽然证书和密钥存放在同一个文件，只有证书会发送给客户端，密钥不会发送。
ssl_protocols和ssl_ciphers指令可以用来强制用户连接只能引入SSL/TLS那些强壮的协议版本和强大的加密算法。从1.0.5版本开始，nginx默认使用“ssl_protocols SSLv3 TLSv1”和“ssl_ciphers HIGH:!aNULL:!MD5”，所以只有在之前的版本，明确地配置它们才是有意义的。从1.1.13和1.0.12版本开始，nginx默认使用“ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2”。

CBC模式的加密算法容易受到一些攻击，尤其是BEAST攻击（参见CVE-2011-3389）。可以通过下面配置调整为优先使用RC4-SHA加密算法::

    ssl_ciphers RC4:HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

5.2 HTTPS服务器优化

SSL操作需要消耗CPU资源，所以在多处理器的系统，需要启动多个工作进程，而且数量需要不少于可用CPU的个数。最消耗CPU资源的SSL操作是SSL握手，有两种方法可以将每个客户端的握手操作数量降到最低：第一种是保持客户端长连接，在一个SSL连接发送多个请求，第二种是在并发的连接或者后续的连接中重用SSL会话参数，这样可以避免SSL握手的操作。会话缓存用于保存SSL会话，这些缓存在工作进程间共享，可以使用ssl_session_cache指令进行配置。1M缓存可以存放大约4000个会话。默认的缓存超时是5分钟，可以使用ssl_session_timeout加大它。
下面是一个针对4核系统的配置优化的例子，使用10M的共享会话缓存::

    worker_processes  4;

    http {
        ssl_session_cache    shared:SSL:10m;
        ssl_session_timeout  10m;
    
        server {
            listen              443;
            server_name         www.example.com;
            keepalive_timeout   70;
    
            ssl                 on;
            ssl_certificate     www.example.com.crt;
            ssl_certificate_key www.example.com.key;
            ssl_protocols       SSLv3 TLSv1 TLSv1.1 TLSv1.2;
            ssl_ciphers         HIGH:!aNULL:!MD5;
            ...

5.3 SSL证书链

有些浏览器不接受那些众所周知的证书认证机构签署的证书，而另外一些浏览器却接受它们。这是由于证书签发使用了一些中间认证机构，这些中间机构被众所周知的证书认证机构授权代为签发证书，但是它们自己却不被广泛认知，所以有些客户端不予识别。针对这种情况，证书认证机构提供一个证书链的包裹，用来声明众所周知的认证机构和自己的关系，需要将这个证书链包裹与服务器证书合并成一个文件。在这个文件里，服务器证书需要出现在认证方证书链的前面::

    $ cat www.example.com.crt bundle.crt > www.example.com.chained.crt

这个文件需要使用ssl_certificate指令来引用::

    server {
        listen              443;
        server_name         www.example.com;
        ssl                 on;
        ssl_certificate     www.example.com.chained.crt;
        ssl_certificate_key www.example.com.key;
        ...
    }

如果服务器证书和认证方证书链合并时顺序弄错了，nginx就不能正常启动，而且会显示下面的错误信息::

    SSL_CTX_use_PrivateKey_file(" ... /www.example.com.key") failed
       (SSL: error:0B080074:x509 certificate routines:
        X509_check_private_key:key values mismatch)

因为nginx首先需要用私钥去解密服务器证书，而遇到的却是认证方的证书。

浏览器通常会将那些被受信的认证机构认证的中间认证机构保存下来，那么这些浏览器以后在遇到使用这些中间认证机构但不包含证书链的情况时，因为已经保存了这些中间认证机构的信息，所以不会报错。可以使用openssl命令行工具来确认服务器发送了完整的证书链::

    $ openssl s_client -connect www.godaddy.com:443
    ...
    Certificate chain
     0 s:/C=US/ST=Arizona/L=Scottsdale/1.3.6.1.4.1.311.60.2.1.3=US
         /1.3.6.1.4.1.311.60.2.1.2=AZ/O=GoDaddy.com, Inc
         /OU=MIS Department/CN=www.GoDaddy.com
         /serialNumber=0796928-7/2.5.4.15=V1.0, Clause 5.(b)
       i:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
         /OU=http://certificates.godaddy.com/repository
         /CN=Go Daddy Secure Certification Authority
         /serialNumber=07969287
     1 s:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
         /OU=http://certificates.godaddy.com/repository
         /CN=Go Daddy Secure Certification Authority
         /serialNumber=07969287
       i:/C=US/O=The Go Daddy Group, Inc.
         /OU=Go Daddy Class 2 Certification Authority
     2 s:/C=US/O=The Go Daddy Group, Inc.
         /OU=Go Daddy Class 2 Certification Authority
       i:/L=ValiCert Validation Network/O=ValiCert, Inc.
         /OU=ValiCert Class 2 Policy Validation Authority
         /CN=http://www.valicert.com//emailAddress=info@valicert.com
    ...

在这个例子中，www.GoDaddy.com的服务器证书（#0）的受签者（“s”）是被签发机构（“i”）签名的，而这个签发机构又是证书（#1）的受签者，接着证书（#1）的签发机构又是证书（#2）的受签者，最后证书（#2）是被众所周知的签发机构ValiCert, Inc签发。ValiCert, Inc的证书内嵌在浏览器中，被浏览器自动识别（这段话神似英国诗《在Jack盖的房子里》里面的内容）。

如果没有加入认证方证书链，就只会显示服务器证书（#0）。

5.4 合并HTTP/HTTPS主机

如果HTTP和HTTPS虚拟主机的功能是一致的，可以配置一个虚拟主机，既处理HTTP请求，又处理HTTPS请求。 配置的方法是删除ssl on的指令，并在*:443端口添加参数ssl::

    server {
        listen              80;
        listen              443 ssl;
        server_name         www.example.com;
        ssl_certificate     www.example.com.crt;
        ssl_certificate_key www.example.com.key;
        ...
    }

.. note::
    
    在0.8.21版本以前，只有添加了default参数的监听端口才能添加ssl参数：
    ``listen  443  default ssl;``

5.5 基于名字的HTTPS主机

如果在同一个IP上配置多个HTTPS主机，会出现一个很普遍的问题::

    server {
    listen          443;
    server_name     www.example.com;
    ssl             on;
    ssl_certificate www.example.com.crt;
    ...
    }
    
    server {
        listen          443;
        server_name     www.example.org;
        ssl             on;
        ssl_certificate www.example.org.crt;
        ...
    }

使用上面的配置，不论浏览器请求哪个主机，都只会收到默认主机www.example.com的证书。这是由SSL协议本身的行为引起的——先建立SSL连接，再发送HTTP请求，所以nginx建立SSL连接时不知道所请求主机的名字，因此，它只会返回默认主机的证书。

最古老的也是最稳定的解决方法就是每个HTTPS主机使用不同的IP地址::

    server {
    listen          192.168.1.1:443;
    server_name     www.example.com;
    ssl             on;
    ssl_certificate www.example.com.crt;
    ...
    }
    
    server {
        listen          192.168.1.2:443;
        server_name     www.example.org;
        ssl             on;
        ssl_certificate www.example.org.crt;
        ...
    }

5.6 带有多个主机名的SSL证书

也有其他一些方法可以实现多个HTTPS主机共享一个IP地址，但都有不足。其中一种方法是使用在“SubjectAltName”字段中存放多个名字的证书，比如www.example.com和www.example.org。但是，“SubjectAltName”字段的长度有限制。

另一种方式是使用主机名中含有通配符的证书，比如*.example.org。这个证书匹配www.example.org，但是不匹配example.org和www.sub.example.org。这两种方法可以结合在一起——使用在“SubjectAltName”字段中存放的多个名字的证书，这些名字既可以是确切的名字，也可以是通配符，比如example.org和*.example.org。

最好将带有多个名字的证书和它的密钥文件配置在http配置块中，这样可以只保存一份内容拷贝，所有主机的配置都从中继承::

    ssl_certificate      common.crt;
    ssl_certificate_key  common.key;
    
    server {
        listen          443;
        server_name     www.example.com;
        ssl             on;
        ...
    }
    
    server {
        listen          443;
        server_name     www.example.org;
        ssl             on;
        ...
    }


5.7 主机名指示

在一个IP上运行多个HTTPS主机的更通用的方案是TLS主机名指示扩展（SNI，RFC6066），它允许浏览器和服务器进行SSL握手时，将请求的主机名传递给服务器，因此服务器可以知道使用哪一个证书来服务这个连接。但SNI只得到有限的浏览器的支持。下面列举支持SNI的浏览器最低版本和平台信息:

    * Opera 8.0；
    * MSIE 7.0（仅在Windows Vista操作系统及后续操作系统）；
    * Firefox 2.0和使用Mozilla平台1.8.1版本的其他浏览器；
    * Safari 3.2.1（Windows版需要最低Vista操作系统）；
    * Chrome（Windows版需要最低Vista操作系统）。

.. note::
    
    通过SNI只能传递域名，但是，当请求中包含可读的IP地址时，某些浏览器将服务器的IP地址作为服务器的名字进行了传送。这是一个错误，大家不应该依赖于这个。

为了在nginx中使用SNI，那么无论是在编译nginx时使用的OpenSSL类库，还是在运行nginx时使用的OpenSSL运行库，都必须支持SNI。从0.9.8f版本开始，OpenSSL通过“--enable-tlsext”配置选项加入SNI支持，从0.9.8j版本开始，此选项成为默认选项。当nginx被编译成支持SNI时，在使用“-V”选项运行时会显示如下信息::

    $ nginx -V
    ...
    TLS SNI support enabled
    ...

但是，当开启SNI支持的nginx被动态链接到不支持SNI的OpenSSL库上时，nginx会显示如下警告::

    nginx was built with SNI support, however, now it is linked
    dynamically to an OpenSSL library which has no tlsext support,
    therefore SNI is not available

5.8 兼容性

    * 从0.8.21和0.7.62版本开始，使用“-V”选项运行nginx时，将显示SNI支持状态信息。
    * 从0.7.14版本开始，listen指令支持ssl参数。
    * 从0.5.32版本开始，支持SNI。
    * 从0.5.6版本开始，支持SSL会话缓存，并可在工作进程间共享。
    * 0.7.65、0.8.19及以后版本，默认SSL协议是SSLv3、TLSv1、TLSc1.1和TLSv1.2（如果OpenSSL库支持）。
    * 0.7.64、0.8.18及以前版本，默认SSL协议是SSLv2、SSLv3和TLSv1。
    * 1.0.5及以后版本，默认SSL密码算法是HIGH:!aNULL:!MD5。
    * 0.7.65、0.8.20及以后版本，默认SSL密码算法是HIGH:!ADH:!MD5。
    * 0.8.19版本，默认SSL密码算法是ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM。
    * 0.7.64、0.8.18及以前版本，默认SSL密码算法是ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP。
