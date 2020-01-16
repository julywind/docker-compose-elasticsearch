为了方便操作，给docker-compose 设置一个别名
 alias dc=docker-compose
 
启动集群：dc up -d

elastic 7.x以后 内置了x-pack, 项目默认没有打开x-pack认证
修改elasticsearch.yml:
 xpack.security.enabled: true
 
#然后重启项目
dc restart

#进入主节点
dc exec es01 bash
#自动生成默认密码，记录下来
elasticsearch-setup-password -v auto
#也可以手动生成密码
#elasticsearch-setup-password -v interactive

#生成证书
elasticsearch-certutil cert -out config/elastic-certificates.p12 -pass ""

#导出证书： 暂时可以copy到data目录下, 因为data目录挂在了外部路径
cp config/elastic-certificates.p12 ./data
#退出容器es01
exit

mkdir ssl
cp -a es7-data/elastic-certificates.p12 ./ssl

#修改docker-compose.yml，去掉证书挂载配置项的注释，es01和es02两个都需要改
#修改kibana.yml 使用前面设置的kibana账号的密码，替换配置文件内的占位符，并去掉注释
#修改elasticsearch.yml， 去掉xpack.security.transport.ssl这几项前的注释符

#重新启动全部服务
dc down && dc up -d

#重启，检查
http://localhost:9200  应该就需要输入用户名密码了

ps: 凡是弹出让登录的地方 都应该使用用户名为elastic的那一套


生成证书部分详细记录：
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
如果docker-compose启动后机器出现卡死：
则需要检查一下剩余内存，至少1g

elk启动的时候可能会提示如下错误:
1. max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
原因以及解决办法：
运行elasticsearch需要vm.max_map_count至少需要262144内存
切换到root用户修改配置sysctl.conf
vi /etc/sysctl.conf
在尾行添加以下内容   
vm.max_map_count=262144
并执行命令
sysctl -p
