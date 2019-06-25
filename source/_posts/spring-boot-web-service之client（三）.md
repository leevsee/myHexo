---
title: spring boot web service之client（三）
date: 2018-07-22 20:25:13
tags:
---

喵喵喵，web service目前就差客户端了。那我就不分篇，spring的客户端和java客户端。  

首先是 **spring客户端** 

---

## 1.添加依赖

``` 
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web-services</artifactId>
</dependency>
```

## 2.根据WSDL生产JavaCode

这里其实可以把服务端的`JavaCode`复制过来，当然也可以自己生成。
首先配置`maven jaxb`，用来生成wsdl的`Java Code`
``` 
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
		<plugin>
			<groupId>org.jvnet.jaxb2.maven2</groupId>
			<artifactId>maven-jaxb2-plugin</artifactId>
			<version>0.14.0</version>
			<executions>
				<execution>
					<goals>
						<goal>generate</goal>
					</goals>
				</execution>
			</executions>
			<configuration>
				<schemaLanguage>WSDL</schemaLanguage>
				<generatePackage>com.leeves.wsrnewdemo.ws.leeves</generatePackage>
				<generateDirectory>${basedir}/src/main/java</generateDirectory>
				<schemas>
					<schema>
						<!--<fileset>-->
						<!--&lt;!&ndash; Defaults to schemaDirectory. &ndash;&gt;-->
						<!--<directory>${basedir}/src/main/resources/schemas</directory>-->
						<!--&lt;!&ndash; Defaults to schemaIncludes. &ndash;&gt;-->
						<!--<includes>-->
						<!--<include>*.wsdl</include>-->
						<!--</includes>-->
						<!--&lt;!&ndash; Defaults to schemaIncludes &ndash;&gt;-->
						<!--&lt;!&ndash;<excludes>&ndash;&gt;-->
						<!--&lt;!&ndash;<exclude>*.xs</exclude>&ndash;&gt;-->
						<!--&lt;!&ndash;</excludes>&ndash;&gt;-->
						<!--</fileset>-->
						<url>http://localhost:7007/ws/leevesService.wsdl</url>
					</schema>
				</schemas>
			</configuration>
		</plugin>
	</plugins>
</build>
```
PS：这里我是根据url地址生成，也可以先把wsdl保存下来，然后根据文件生成  


然后`mvn install`生成即可。
![enter description here][1]

生成后的代码如下：
![enter description here][2]


## 3.ws客户端配置
编写发送方法：

``` 
public class WSClient extends WebServiceGatewaySupport {

    public LeevesServiceResponse sendToLeevesService(String reqMsg) {
        LeevesServiceRequest request = new LeevesServiceRequest();
        request.setReqMsg(reqMsg);
        return (LeevesServiceResponse) getWebServiceTemplate().marshalSendAndReceive(
                "http://localhost:7007/ws/leevesService.wsdl", request);
    }

    public TestServiceResponse sendToTestService(String reqMsg) {
        TestService request = new TestService();
        request.setReqMsg(reqMsg);
        return (TestServiceResponse) getWebServiceTemplate().marshalSendAndReceive(
                "http://localhost:7007/ws/testService.wsdl", request);
    }

}
```

添加解析路径和url，并放到spring容器中：

``` 
@Configuration
public class WSConfig {

    @Bean
    public Jaxb2Marshaller leevesMarshaller() {
        Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
        marshaller.setContextPath("com.leeves.wsrnewdemo.ws.leeves");
        return marshaller;
    }

    @Bean
    public Jaxb2Marshaller testMarshaller() {
        Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
        marshaller.setContextPath("com.leeves.wsrnewdemo.ws.test");
        return marshaller;
    }

    @Bean
    public WSClient leevesWSClient(Jaxb2Marshaller leevesMarshaller) {
        WSClient client = new WSClient();
        client.setDefaultUri("http://localhost:7007/ws/leevesService.wsdl");
        client.setMarshaller(leevesMarshaller);
        client.setUnmarshaller(leevesMarshaller);
        return client;
    }

    @Bean
    public WSClient testWSClient(Jaxb2Marshaller testMarshaller) {
        WSClient client = new WSClient();
        client.setDefaultUri("http://localhost:7007/ws/testService.wsdl");
        client.setMarshaller(testMarshaller);
        client.setUnmarshaller(testMarshaller);
        return client;
    }

}
```

## 4.测试通过
噔噔，完美通过
![enter description here][3]



<br/>
<br/>
<br/>
<br/>
----------

**接下来是用jdk自带的生成客户端**

## 1.添加依赖

![enter description here][4]
不存在的。。本身jdk自带要什么鬼依赖


## 2.生成wsdl客户端

在F盘创建wsdl文件夹，然后命令行输入：

``` 
wsimport -s F:\wsdl -p com.leeves.wsjwsrnewdemo.webservice http://localhost:7000/jws/leevesService?wsdl
// F:\wsdl	生成的到磁盘地址
// com.leeves.wsjwsrnewdemo.webservice	包路径
// http://localhost:7007/ws/leevesService.wsdl	wsdl地址
```
![enter description here][5]

把生成的文件放到项目中

## 3.ws客户端配置

``` 
public class WSClient {

    private static final String CONNECTION_TIMEOUT = "javax.xml.hisws.client.connectionTimeout";

    private static final String RECEIVE_TIMEOUT = "javax.xml.hisws.client.receiveTimeout";
    //5秒
    private static long timeOut = 5 * 1000;

    private static final String WS_URL = "http://172.16.11.32:7000/jws/leevesService?wsdl";

    private static LeevesService leevesService = null;

    public static String getInstance(String reqMsg) {

        if (leevesService == null) {
            LeevesServiceImplService leevesServiceImplService;
            try {
                leevesServiceImplService = new LeevesServiceImplService(new URL(WS_URL));
                leevesService = leevesServiceImplService.getLeevesServiceImplPort();
            } catch (MalformedURLException e) {
                e.printStackTrace();
            }
            Assert.notNull(leevesService, "web service client instance must not null!");
            ((BindingProvider) leevesService).getRequestContext().put(CONNECTION_TIMEOUT, timeOut);
            ((BindingProvider) leevesService).getRequestContext().put(RECEIVE_TIMEOUT, timeOut);
        }

        return leevesService.leevesService(reqMsg);
    }

}
```

## 4.测试
直接调用，就有数据鸟
![enter description here][6]



---
最后，总结一下，客户端来说还是用jdk生成的客户端方便。

附上两个代码地址：  
https://github.com/leevsee/ws-r-new-demo  
https://github.com/leevsee/ws-jws-r-new-demo




[1]: http://lixin.piaozu.com.cn/ws_r_mvn_install.png
[2]: http://lixin.piaozu.com.cn/ws_r_leeves_test.png
[3]: http://lixin.piaozu.com.cn/ws_r_test.png
[4]: http://lixin.piaozu.com.cn/ws_r_no.jpg
[5]: http://lixin.piaozu.com.cn/ws_r_wsimport.png
[6]: http://lixin.piaozu.com.cn/ws_r_jws_test.png


