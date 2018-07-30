---
title: spring-boot-web-service之ws（一）
date: 2018-07-18 23:20:45
tags:
---

题外话：ε=(´ο｀*)))日渐消瘦，现在感觉写博客的心情也少了些。嘛，最近看了个视频深有感触。是时候记录一下自己的时间，从而才能够更好的自觉。  
背景：
	目前web service有两类实现：
	一：jws+cxf  
	二：通过spring ws来实现
其实，想先来一篇jws+cxf。但是spring ws相对复杂些，那就先上复杂的好了，就是那么任性╮(╯▽╰)╭  
吐槽一下，百度上面的，全是官方文档的例子。类名和operation方法都是一模一样。很多时间不能满足业务需求。本篇除了会有官方的例子，还会有自定义的web serivce。

## 1.添加依赖

``` 

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web-services</artifactId>
</dependency>
<dependency>
	<groupId>wsdl4j</groupId>
	<artifactId>wsdl4j</artifactId>
</dependency>
```

## 2.创建XML Schema文件  
用于生成wsdl

``` 
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:tns="http://webservice.leeves.com/ws"
           targetNamespace="http://webservice.leeves.com/ws" elementFormDefault="qualified">

    <xs:element name="leevesServiceRequest">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="reqMsg" type="xs:string"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

    <xs:element name="leevesServiceResponse">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="resMsg" type="xs:string"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

</xs:schema>
```

## 3.创建配置类
生成接收返回Request和Response类  
我是使用idea，选中Schema右键生产
![enter description here](http://7xz8pr.com1.z1.glb.clouddn.com/wsservicejaxb.png)
具体类如下：
![enter description here](http://7xz8pr.com1.z1.glb.clouddn.com/wsserviceleevesService.png)



## 4.创建配置类
获取XsdSchema实例，并转换成wsdl

``` 

@EnableWs
@Configuration
public class WebServiceConfig extends WsConfigurerAdapter {

    /**
     * 把XSD schemas转换成wsdl
     */
    @Bean(name = "leevesService")
    public DefaultWsdl11Definition defaultWsdl11Definition(XsdSchema leevesServiceSchema) {
        DefaultWsdl11Definition wsdl11Definition = new DefaultWsdl11Definition();
        wsdl11Definition.setPortTypeName("LeevesService");
        wsdl11Definition.setLocationUri("/ws");
        wsdl11Definition.setTargetNamespace("http://webservice.leeves.com/ws");
        wsdl11Definition.setSchema(leevesServiceSchema);
        return wsdl11Definition;
    }

    /**
     * 创建leeves的XsdSchema实例
     */
    @Bean
    public XsdSchema leevesServiceSchema() {
        return new SimpleXsdSchema(new ClassPathResource("schema/leevesService.xsd"));
    }

}
```

## 5.发布webservice端口

``` 

@Endpoint
public class WebServiceEndpoint {

    private static final String NAMESPACE_URI = "http://webservice.leeves.com/ws";

    /**
     * 处理web service请求
     * 其中localPart是input的节点
     */
    @PayloadRoot(namespace = NAMESPACE_URI, localPart = "leevesServiceRequest")
    @ResponsePayload
    public LeevesServiceResponse getReqMsg(@RequestPayload LeevesServiceRequest leevesServiceRequest) {
        String reqMsg = leevesServiceRequest.getReqMsg();
        System.out.println("----LeevesService服务端接收到----：" + reqMsg);
        LeevesServiceResponse leevesServiceResponse = new LeevesServiceResponse();
        leevesServiceResponse.setResMsg("LeevesService服务端收到：" + reqMsg + "！向客户端发送数据：我是LeevesService");
        return leevesServiceResponse;
    }

}
```

-  ## 使用soapui测试

添加soap地址：
![enter description here](http://7xz8pr.com1.z1.glb.clouddn.com/wsservicesoapui.png)


发起请求：
![enter description here](http://7xz8pr.com1.z1.glb.clouddn.com/wsservicesoapui_leeves.png)


后台：
![enter description here](http://7xz8pr.com1.z1.glb.clouddn.com/wsserviceleevesService%E6%8E%A5%E5%8F%97.png)

---
---
---
以上就是正常的一个使用spring ws创建服务端的例子。在spring ws中入参和出参均以Request和Response结尾。
![enter description here](http://7xz8pr.com1.z1.glb.clouddn.com/wsserviceleevesService_xsd.png)


那么问题来了，有些webservice的不以Request和Response结尾。肿么办？？？？  

这时候你就需要自定义wsdl了。

下面我们先看看最终效果：
这是上面的做的leeveSerivce：
![enter description here](http://7xz8pr.com1.z1.glb.clouddn.com/wsserviceleevesService_wsdl.png)
这是不以Request结尾的testSerice：
![enter description here](http://7xz8pr.com1.z1.glb.clouddn.com/wsservicetestService_wsdl.png)

那么怎么要来做呢？

## 1.重写MessagesProvider和PortTypesProvider
下面是需要修改的地方

``` 

public class MySuffixBasedMessagesProvider extends DefaultMessagesProvider {

	//其他代码省略
	
    /** 自定义后缀 */
    private String customSuffixStr = "Service";
	
	/**
     * 判断自定义后缀是element元素
     */
    @Override
    protected boolean isMessageElement(Element element) {
        if (super.isMessageElement(element)) {
            String elementName = getElementName(element);
            Assert.hasText(elementName, "Element has no name");
            return elementName.endsWith(getRequestSuffix()) || elementName.endsWith(getResponseSuffix()) ||
                    elementName.endsWith(getFaultSuffix()) || elementName.endsWith(getCustomSuffixStr());
        }
        else {
            return false;
        }
    }

	//其他代码省略

}
```

``` 

public class MySuffixBasedPortTypesProvider extends AbstractPortTypesProvider {

	//其他代码省略
	
    /** 自定义后缀 */
    private String customSuffixStr = "Service";
	
    /**
     * 添加自定义后缀到OperationName
     */
    @Override
    protected String getOperationName(Message message) {
        String messageName = getMessageName(message);
        if (messageName != null) {
            if (messageName.endsWith(getCustomSuffixStr())) {
                return messageName;
            } else if (messageName.endsWith(getRequestSuffix())) {
                return messageName.substring(0, messageName.length() - getRequestSuffix().length());
            } else if (messageName.endsWith(getResponseSuffix())) {
                return messageName.substring(0, messageName.length() - getResponseSuffix().length());
            } else if (messageName.endsWith(getFaultSuffix())) {
                return messageName.substring(0, messageName.length() - getFaultSuffix().length());
            }
        }
        return null;
    }

	//其他代码省略

}
```

## 2.自定义Wsdl11Definition
把上面重写的MessagesProvider和PortTypesProvider，放到Wsdl11Definition类中

``` 

public class MyWsdl11Definition  implements Wsdl11Definition, InitializingBean {

	//其他代码省略
	
    /** 添加定义MessagesProvider */
    private final MySuffixBasedMessagesProvider messagesProvider = new MySuffixBasedMessagesProvider();
    /** 添加定义PortTypesProvider */
    private final MySuffixBasedPortTypesProvider portTypesProvider = new MySuffixBasedPortTypesProvider();
	
	//其他代码省略

}
```

## 3.web service 配置

``` 

@EnableWs
@Configuration
public class WebServiceConfig extends WsConfigurerAdapter {

    /**
     * 把XSD schemas转换成wsdl
     */
    @Bean(name = "testService")
    public MyWsdl11Definition mMyWsdl11Definition(XsdSchema testServiceSchema) {
        MyWsdl11Definition wsdl11Definition = new MyWsdl11Definition();
        wsdl11Definition.setPortTypeName("TestService");
        wsdl11Definition.setLocationUri("/ws");
        wsdl11Definition.setTargetNamespace("http://webservice.leeves.com/ws");
        wsdl11Definition.setSchema(testServiceSchema);
        return wsdl11Definition;
    }
	
    /**
     * 创建test的XsdSchema实例
     */
    @Bean
    public XsdSchema testServiceSchema() {
        return new SimpleXsdSchema(new ClassPathResource("schema/testService.xsd"));
    }	

}
```

-  ## 使用soapui测试
完美通过，^.^，下次再带来spring boot ws client客户端
![enter description here](http://7xz8pr.com1.z1.glb.clouddn.com/wsservicetestService_soapui.png)



忘了忘了，补上代码地址：https://github.com/leevsee/ws-new-demo