# 服务接入配置中心说明

目前只支持springboot与springcloud服务

接入之前先在服务端管理后台添加对应的配置。配置内容，分为全局配置（global config）和应用配置(application config)。全局配置指的是所有应用都是共用的配置，但它的优先及低于应用配置。一个运行环境下会有一个全局配置。

## 接入步骤

### 1. 引入配置中心客户端

```xml
<dependency>
    <groupId>com.kj.config-keeper</groupId>
    <artifactId>kj-config-client</artifactId>
    <version>1.0.0</version>
</dependency>
```

### 2. 配置 src/main/resources/bootstrap.yml

以下内容是必须配置的：

```yaml
spring:
 application:
   name: item        # 设置应用名称，不能与其它应用重名，必须配置
 profiles:
   active: ${profile:dev}          # 设置生效的profile, 必须配置
kj:
 config:
   cache-path: ${apphome:.}/config/

# 启用spring boot的 endpoints，用于监控，刷新配置等
management:
 endpoints:
   web:
     base-path: /ops    #监控路径
     exposure:
       include: '*'    
```

通常应用是运行在不同的环境中，比如：开发环境(dev)、测试环境(test)、RC环境(rc)、生产环境(prod)。那么需要根据不同的环境配置不同的配置中心服务地址，比如，开发环境的配置，配置到src/main/resources/bootstrap-dev.yml文件中：

```yaml
kj:
 config:
   enabled: true                 # 是否启用配置中心，默认值为：true;
   uris:
   - http://127.0.0.1:8080/       # 配置中心服务地址 服务集群部署则地址配置多个
   username: admin                # 调用接口用户名(非配置中心登录的用户名和密码)
   password: 123456               # 调用接口密码
   cachePath: ./config            # 配置缓存路径，默认值为：./config
   cacheTimeOut: 0                # 本地缓存过期时间(单位：秒),如果小于等于0时，一直有效
   failFast: false                # 是否快速失败，如果为true时，当访问配置中心时立即抛异常；如果为false时，会尝试加载3次，并会尝试获取本地缓存，最终还没有配置，才会抛异常。默认值：false
```

测试环境的配置文件为：src/main/resources/bootstrap-test.yml、RC环境的配置文件为：src/main/resources/bootstrap-rc.yml、生产环境的配置文件为：src/main/resources/bootstrap-prod.yml 参考开发环境的配置进行设置。

### 3. 激活profile

上面通过bootstrap-dev.yml、bootstrap-test.yml、bootstrap-rc.yml、bootstrap-prod.yml等文件来切换不同环境的配置，但需要通过设置spring.profiles.active来激活。

激活Spring boot profile的方式很多，推荐使用jvm参数指定：

```
java -jar xxx.jar --profile=prod
```

### 4. java配置代码实例

spring boot中读取配置项内容的方法很多，最常用的有两种方式使用:@Value 和 @ConfigurationProperties

例如：



```java
@RestController
@RefreshScope  // 如果不加此，下面的@Value 无法刷新
public class ConfigTestController {

    @Value("${configSwitch:false}")
    private boolean configSwitch = false;
    
    @Autowired
    private DefaultUserWapper defaultUserWapper;

    @GetMapping({ "/", "index" })
    public String getConfig() {
        return "--" + configSwitch+";defaultUserWapper-->"+defaultUserWapper;
   }
}
```

使用 @Value 读取配置内容，如果要实现动态刷新的话，需要结合@RefreshScope一起使用。

[@RefreshScope官方说明](http://cloud.spring.io/spring-cloud-static/Edgware.SR2/single/spring-cloud.html#_refresh_scope)



# 参考资料

<https://github.com/sxfad/config-keeper>