### step1
同样的配置文件, 同样的jar包, 在测试环境中可以成功注册到nacos中, 但是无法注册到生产环境中



### step2
查看了nacos客户端日志, 发现是有执行nacos registry的.
查看了nacos服务端的naming日志, 发现也是有stocks-service的注册的(另外没有看到gateway-service的注册日志, 反而gateway-service成功注册到了nacos中)


### step3
通过nacos的openapi手动注册, 成功注册, 并能通过健康检查:
```sh
获取token:
curl -X POST '127.0.0.1:8848/nacos/v1/auth/login' -d 'username=nacos&password=nacos##6677'


注册实例:
curl -d 'namespaceId=trunk_test-namespace-id' -d 'groupName=prod' -d 'serviceName=stocks-service' -d 'ip=10.129.133.192' -d 'port=8890' -d 'clusterName=SH' -d 'ephemeral=false' -X POST 'http://10.129.2.42:8848/nacos/v2/ns/instance' -H "Authorization: Bearer eyJhbGciOiJIUzM4NCJ9.eyJzdWIiOiJuYWNvcyIsImV4cCI6MTczOTE4NjczMX0.MBa3LRMNVSjn469q9MSMZng4wucY1DQHLR5TlYLOcbZHtCcSsWZSznL5C0op1xj_"


注销实例:
curl -d 'namespaceId=trunk_test-namespace-id' -d 'groupName=prod' -d 'serviceName=stocks-service' -d 'ip=10.129.133.192' -d 'port=8890' -d 'clusterName=SH' -d 'ephemeral=false' -X DELETE 'http://10.129.2.42:8848/nacos/v2/ns/instance' -H "Authorization: Bearer eyJhbGciOiJIUzM4NCJ9.eyJzdWIiOiJuYWNvcyIsImV4cCI6MTczOTE4NjczMX0.MBa3LRMNVSjn469q9MSMZng4wucY1DQHLR5TlYLOcbZHtCcSsWZSznL5C0op1xj_"
```



