第一步：切换到elastsearch的目录下，使用下列命令生成证书

```bin/elasticsearch-certutil cert -out config/elastic-certificates.p12 -pass ""```

第二步：打开config/elasticsearch.yaml，在尾部添加下面一行代码：

```
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```
