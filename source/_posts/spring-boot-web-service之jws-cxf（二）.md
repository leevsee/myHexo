---
title: spring-boot-web-service之jws+cxf（二）
date: 2018-07-20 00:22:55
tags:
---

今天来点简单的，使用javax.jws+cxf来发布web service。  
之前因为spring boot 2新出的时候，不支持cxf，所以才去研究一下spring自身的ws。  
现在cxf有新版本了，也支持spring boot2。所以补一个例子好啦。  

## 1.添加依赖

``` 
<dependency>
	<groupId>org.apache.cxf</groupId>
	<artifactId>cxf-spring-boot-starter-jaxws</artifactId>
	<version>3.2.5</version>
</dependency>
```

## 2.创建webservice接口和实现类  

``` 
@WebService
public interface LeevesService {

    @WebMethod
    String leevesService(String reqMsg);
}
```

``` 
@Slf4j
@WebService(targetNamespace = "http://webservice.wsjwsnewdemo.leeves.com/", endpointInterface = "com.leeves.wsjwsnewdemo.webservice.LeevesService")
public class LeevesServiceImpl implements LeevesService {

    @Override
    public String leevesService(String reqMsg) {
        log.info("---jws leevesService reqMsg---:" + reqMsg);
        return "---jws leevesService resMsg---:" + reqMsg;
    }

}
```

## 3.配置cxf  

``` 
@Configuration
public class CXFCofnig {

    @Bean
    public ServletRegistrationBean<CXFServlet> cxfDispatcherServlet() {
        return new ServletRegistrationBean<>(new CXFServlet(), "/jws/*");
    }

    @Bean(name = Bus.DEFAULT_BUS_ID)
    public SpringBus springBus() {
        return new SpringBus();
    }

    @Bean
    public Endpoint endpoint() {
        EndpointImpl endpoint = new EndpointImpl(springBus(), new LeevesServiceImpl());
        endpoint.publish("/leevesService");
        return endpoint;
    }

}
```

然后。。。就完了，是不是很简单！！！比spring ws简单多了。
这个是wsdl页面：
![enter description here][1]
-  ## 使用soapui测试
![enter description here][2]

--- 
--- 
---

最后，总结，相比较jws比较简单，没有像像ws那样可以自定义wsdl。一般来说，用jws就够鸟。

web service服务端发完了。接下来，再来一篇web service的客户端。这个系列就算完结了。

最后上代码地址：https://github.com/leevsee/ws-jws-new-demo



[1]: http://lixin.piaozu.com.cn/jws_leevesService.png
[2]: http://lixin.piaozu.com.cn/jws_soapui.png

