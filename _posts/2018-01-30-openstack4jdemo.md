---
layout: post
title:  "openstack4j集成Openstack问题及样例代码"
data:   2017-09-06 14:31:48
categories: openstack
tags: openstack openstack4j 云计算
---


## 参考资料
- 官方地址：http://www.openstack4j.com/
- 发布包地址：http://search.maven.org/
- github地址:https://github.com/ContainX/openstack4j/

## 集成步骤
分为mavan集成方式和直接集成方式，这里采用直接集成的方式，其他集成方式看官方文档。
## 版本支持
* OpenStack4j 3.0.X - Java 7 (JDK 8 preferred)
* OpenStack4j 2.0.X - Java 7

## 集成代码参考

```
import java.util.List;

import org.openstack4j.api.OSClient.OSClientV2;
import org.openstack4j.api.OSClient.OSClientV3;
import org.openstack4j.model.common.Identifier;
import org.openstack4j.model.compute.Server;
import org.openstack4j.model.identity.v3.User;
import org.openstack4j.model.image.Image;
import org.openstack4j.openstack.OSFactory;


import org.openstack4j.api.OSClient.OSClientV2;
import org.openstack4j.api.OSClient.OSClientV3;
import org.openstack4j.model.common.Identifier;
import org.openstack4j.model.compute.Server;
import org.openstack4j.model.identity.v3.User;
import org.openstack4j.model.image.Image;
import org.openstack4j.openstack.OSFactory;

/**
 * Created by zhangqk on 2017/8/24.
 */
public class Test {

    OSClientV2 clientV2 = null;
    OSClientV3 clientV3 = null;
    /**
     * @param args
     */
    public static void main(String[] args) {
//        OSClientV2 os = new Test().getClientV2();
        OSClientV2 os = new Test().getClientV2100();
//		OSClientV3 os = new Test().getClientV3();
//		List<Server> servers = (List<Server>) os.compute().servers().list();
        List<Image> images = (List<Image>) os.images().list();
        for (Image image : images) {
            System.out.println("镜像名称："+image.getName());
        }
//		if(servers.size() > 0){
//			for(Server server:servers){
//				System.out.println("主机名字："+server.getName());
//			}
//		}
        System.out.println("finsh");
    }

    public OSClientV2 getClientV2(){
        if(clientV2 == null){
            clientV2 = OSFactory.builderV2()
//                    .endpoint("http://10.10.110.100:5000/v2.0")
//                    .credentials("admin","admin")
//                    .tenantName("admin")
//                    .authenticate();
                    .endpoint("http://192.168.56.122:5000/v2.0")
                    .credentials("admin","password")
                    .tenantName("admin")
                    .authenticate();
        }
        return clientV2;
    }

    public OSClientV2 getClientV2100(){
        if(clientV2 == null){
            clientV2 = OSFactory.builderV2()
//                    .endpoint("http://10.10.110.100:5000/v2.0")
//                    .credentials("admin","admin")
//                    .tenantName("admin")
//                    .authenticate();
                    .endpoint("http://192.168.56.122:5000/v2.0")
                    .credentials("admin","password")
                    .tenantName("admin")
                    .authenticate();
        }
        return clientV2;
    }

    public OSClientV3 getClientV3(){
        Identifier domainIdentifier = Identifier.byId("default");
        if (clientV3 == null) {
            OSFactory.enableHttpLoggingFilter(true);
            clientV3 = OSFactory.builderV3()
//                    .endpoint("http://openstack_vip:5000/v3")
                    .endpoint("http://192.168.56.122:5000/v3")
//					.endpoint("http://10.10.110.87:35357/v3")
                    .credentials("admin","password",domainIdentifier)
                    .authenticate();
        }
        return clientV3;
    }


}

```


## 问题及解决
### java-jdk版本不支持
#### 错误描述

```
Exception in thread "main" java.lang.UnsupportedClassVersionError: org/openstack4j/model/common/Identifier : Unsupported major.minor version 51.0
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClassCond(ClassLoader.java:631)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:615)
	at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:141)
	at java.net.URLClassLoader.defineClass(URLClassLoader.java:283)
	at java.net.URLClassLoader.access$000(URLClassLoader.java:58)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:197)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:190)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:306)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:301)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:247)
	at Test.testv3(Test.java:38)
	at Test.main(Test.java:24)
```

#### 解决办法
使用jdk1.8版本

以下是各版本对应的小版本号：
- Java SE 9 = 53 (0x35 hex),
- Java SE 8 = 52 (0x34 hex),
- Java SE 7 = 51 (0x33 hex),
- Java SE 6.0 = 50 (0x32 hex),
- Java SE 5.0 = 49 (0x31 hex),
- JDK 1.4 = 48 (0x30 hex),
- JDK 1.3 = 47 (0x2F hex),
- JDK 1.2 = 46 (0x2E hex),
- JDK 1.1 = 45 (0x2D hex).


### java-jdk版本不支持
#### 错误描述
```
Exception in thread "main" java.lang.NoClassDefFoundError: com/google/common/collect/Lists
	at org.openstack4j.openstack.identity.v3.domain.KeystoneAuth$AuthIdentity.<init>(KeystoneAuth.java:105)
	at org.openstack4j.openstack.identity.v3.domain.KeystoneAuth$AuthIdentity.createCredentialType(KeystoneAuth.java:119)
	at org.openstack4j.openstack.identity.v3.domain.KeystoneAuth.<init>(KeystoneAuth.java:52)
	at org.openstack4j.openstack.client.OSClientBuilder$ClientV3.authenticate(OSClientBuilder.java:165)
	at org.openstack4j.openstack.client.OSClientBuilder$ClientV3.authenticate(OSClientBuilder.java:128)
	at Test.testv3(Test.java:39)
	at Test.main(Test.java:24)
```
#### 解决办法
下载对应的依赖包，包括（2.21，3.0。4）
* 下载地址：https://jar-download.com/?detail_search=a%3A%22openstack4j%22&a=openstack4j
* 


### openstack不是原生的，厂商做了更改，正在解决
#### 错误描述
```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
Exception in thread "main" ConnectionException{message=URI does not specify a valid host name: http:/v1/images/detail, status=0}
	at org.openstack4j.connectors.resteasy.HttpExecutorServiceImpl.invoke(HttpExecutorServiceImpl.java:56)
	at org.openstack4j.connectors.resteasy.HttpExecutorServiceImpl.execute(HttpExecutorServiceImpl.java:30)
	at org.openstack4j.core.transport.internal.HttpExecutor.execute(HttpExecutor.java:51)
	at org.openstack4j.openstack.internal.BaseOpenStackService$Invocation.execute(BaseOpenStackService.java:208)
	at org.openstack4j.openstack.internal.BaseOpenStackService$Invocation.execute(BaseOpenStackService.java:202)
	at org.openstack4j.openstack.image.internal.ImageServiceImpl.list(ImageServiceImpl.java:44)
	at com.sgcc.sgcloud.Test.main(Test.java:26)
Caused by: org.apache.http.client.ClientProtocolException: URI does not specify a valid host name: http:/v1/images/detail
	at org.apache.http.impl.client.AbstractHttpClient.determineTarget(AbstractHttpClient.java:766)
	at org.apache.http.impl.client.AbstractHttpClient.execute(AbstractHttpClient.java:754)
	at org.jboss.resteasy.client.core.executors.ApacheHttpClient4Executor.execute(ApacheHttpClient4Executor.java:182)
	at org.jboss.resteasy.client.ClientRequest.execute(ClientRequest.java:438)
	at org.jboss.resteasy.client.ClientRequest.httpMethod(ClientRequest.java:688)
	at org.jboss.resteasy.client.ClientRequest.httpMethod(ClientRequest.java:694)
	at org.openstack4j.connectors.resteasy.HttpCommand.execute(HttpCommand.java:65)
	at org.openstack4j.connectors.resteasy.HttpExecutorServiceImpl.invokeRequest(HttpExecutorServiceImpl.java:61)
	at org.openstack4j.connectors.resteasy.HttpExecutorServiceImpl.invoke(HttpExecutorServiceImpl.java:54)
	... 6 more
```
#### 解决办法


### xxx
#### 错误描述

```

```

#### 解决办法

### xxx
#### 错误描述

```

```

#### 解决办法