---
title: web service -- spring + cxf 发布web service(2)
date: 2020-01-03 15:17:15
tags: web service
---

这个项目的接口，断断续续做了3个月了，从一开始的一脸懵逼，到现在的一知半解。。。
<!--more-->
## 一、问题描述
项目上是用spring框架加上cxf来发布webservice的，啥都有，就配置一下，再写java就行。
但是，仅仅发布，客户大人是不会满意的，他们对这个报文格式，是有要求滴。
归纳为以下几点:

### 1. 命名空间
以系统简称作为命名空间，并且要求body下每个节点前面都有命名空间前缀
那么我遇到了什么问题呢
首先我们这个系统简称有点长，有6个字母，生成的报文前缀直接给我截取成了3个字母。。。
其次就是每个节点前都要加命名空间前缀，让我好一番折腾

### 2. 节点要求
Body中第一层应使用输入数据模型根节点信息，即服务方法名+"Request"。
而我生成的第一层永远是方法名，第二层才是数据模型。

### 3. Header
服务请求报文在Header中添加服务安全验证信息
想校验安全信息，找不到Header。。。

每每回想起那审查后的十大问题，我都觉得要是到时候报文不严格按照这个来，都对不起我这么久的研究

## 二、解决方法
先上报文，要求是这样的
```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:qwerty="http://qwerty.com">
   <soapenv:Header>
      <qwerty:security>
         <qwerty:username>?</qwerty:username>
         <qwerty:password>?</qwerty:password>
      </qwerty:security>
   </soapenv:Header>
   <soapenv:Body>
	 <qwerty:functionNameRequest>
	    <!--Optional:-->
	    <qwerty:number>?</qwerty:number>
	    <!--Optional:-->
	    <qwerty:id>?</qwerty:id>
	    <!--Optional:-->
         <qwerty:date>?</qwerty:date>
	 </qwerty:functionNameRequest>
   </soapenv:Body>
</soapenv:Envelope>
```
我只能生成这样滴
```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:qwe="http://qwerty.com">
   <soapenv:Header>
      <qwe:security>
         <qwe:username>?</qwe:username>
         <qwe:password>?</qwe:password>
      </qwe:security>
   </soapenv:Header>
   <soapenv:Body>
      <qwe:functionName>
         <qwe:functionNameRequest>
            <!--Optional:-->
            <qwe:paymentReqNumber>111</qwe:paymentReqNumber>
            <!--Optional:-->
            <qwe:contractId>?</qwe:contractId>
            <!--Optional:-->
            <qwe:applyPayDate>?</qwe:applyPayDate>
         </qwe:functionNameRequest>
      </qwe:functionName>
   </soapenv:Body>
</soapenv:Envelope>
```
先说说这样婶儿的是怎么生成的
### 1. 命名空间

#### 1.1. 在接口方法上加注释(这样就会有xmlns，security，request前面会有命名空间)
@WebService(targetNamespace = "http://qwerty.com")
#### 1.2. 在方法参数上加注释(这样方法名前面就会有命名空间)
@XmlElement(required = true, namespace = "http://qwerty.com") @WebParam(name = "functionNameRequest", partName = "functionNameRequest")
                                               FunctionNameRequest functionNameRequest
#### 1.3. 要在对象每个属性前面加命名空间前缀有两个方法
##### 1.3.1. 在属性的get方法上加
@XmlElement(namespace = "http://qwerty.com")
但是必须每个都加，才能实现每个节点都有
##### 1.3.2. 在FunctionNameRequest对象目录下新增文件 package-info.java
```
@XmlSchema(
        namespace = "http://qwerty.com",
        elementFormDefault = XmlNsForm.QUALIFIED,
        xmlns = {
                @XmlNs(prefix = "qwerty", namespaceURI = "http://qwerty.com")
        }
)
package hls.core.ws.dto;

import javax.xml.bind.annotation.XmlNs;
import javax.xml.bind.annotation.XmlNsForm;
import javax.xml.bind.annotation.XmlSchema;
```

### 2. 节点要求
。。。失败了

### 3. Header
在请求接口方法中添加 
@XmlElement(required = true) @WebParam(name = "security", header = true) InHeaderMessage inHeaderMessage
加了@XmlElement(required = true)就不会有被注释了的 Optional: 了。

## 三、拦截器
最后来说说解决一切烦恼的拦截器吧。

### 1. 定义
在 [web service -- spring + cxf 发布web service(1)](http://localhost:4000/2019/11/13/web-service/)
中，已经说过发布接口需要配置cxf-beans.xml，拦截器也在这里配置：
```
    <bean id="inInterceptor" class="hls.core.ws.interceptor.ServiceInReceiveInterceptor"/>
    <bean id="outInterceptor" class="hls.core.ws.interceptor.ServiceOutPreStreamInterceptor"/>
    <jaxws:server id="wsFunctionName" serviceClass="hls.core.ws.service.FunctionName"
                  address="/FunctionName">
        <jaxws:serviceBean>
            <bean class="hls.core.ws.service.impl.FunctionNameImpl"/>
        </jaxws:serviceBean>
        <jaxws:inInterceptors>
            <ref bean="inInterceptor"/>
        </jaxws:inInterceptors>
        <jaxws:outInterceptors>
            <ref bean="outInterceptor"/>
        </jaxws:outInterceptors>
    </jaxws:server>
```
这里配置了两个拦截器，一个输入拦截器，一个输出拦截器。

#### 1.1. 输入阶段
阶段名称|阶段功能描述
--|:--:
RECEIVE|Transport level processing(接收阶段，传输层处理) 
(PRE/USER/POST)_STREAM|Stream level processing/transformations(流处理/转换阶段)
READ|This is where header reading typically occurs(SOAPHeader读取) 
(PRE/USER/POST)_PROTOCOL| Protocol processing, such as JAX-WS SOAP handlers(协议处理阶段，例如JAX-WS的Handler处理)
UNMARSHAL|Unmarshalling of the request(SOAP请求解码阶段)
(PRE/USER/POST)_LOGICAL|Processing of the umarshalled request(SOAP请求解码处理阶段)
PRE_INVOKE|Pre invocation actions(调用业务处理之前进入该阶段)
INVOKE|Invocation of the service(调用业务阶段)
POST_INVOKE|Invocation of the outgoing chain if there is one(提交业务处理结果，并触发输入连接器)

#### 1.2. 输入拦截器
(1) 修改报文：
```
package hls.core.ws.interceptor;

import org.apache.cxf.helpers.IOUtils;
import org.apache.cxf.interceptor.Fault;
import org.apache.cxf.message.Exchange;
import org.apache.cxf.message.Message;
import org.apache.cxf.phase.AbstractPhaseInterceptor;
import org.apache.cxf.phase.Phase;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.StandardCharsets;

/**
 * @Author ever
 * @DATE 2019-12-02 14:14
 * @Description 接口服务端流入拦截器(修改请求报文)
 */
public class ServiceInReceiveInterceptor extends AbstractPhaseInterceptor<Message> {
    protected static Logger log = LoggerFactory.getLogger(ServiceInReceiveInterceptor.class);

    public ServiceInReceiveInterceptor() {
        super(Phase.RECEIVE);
    }

    @Override
    public void handleMessage(Message message) throws Fault {
        InputStream is = message.getContent(InputStream.class);
        if (is != null) {
            try {
                String str = IOUtils.toString(is);
                // 原请求报文
                Exchange exchange = message.getExchange();
                exchange.put("input.content.origin", str);
                log.info("====> request xml=\r\n" + str);

                // 修改报文格式
                str = str.replace("<qwerty:functionName>", 
                "<qwerty:functionName>\n" +
                        "      <qwerty:functionNameRequest>")
                        .replace("</qwerty:functionNameRequest>", 
                        "</qwerty:functionNameRequest>\n" +
                        "</qwerty:functionNameRequest>\n" +
                                "      </qwerty:functionName>")
                        .replace("qwerty", "qwe")
                        .replace("http://qwe.com", "http://qwerty.com");

                // 替换后的报文
                log.info("====> replaced xml=\r\n" + str);
                exchange.put("input.content", str);

                InputStream ism = new ByteArrayInputStream(str.getBytes(StandardCharsets.UTF_8));
                message.setContent(InputStream.class, ism);

            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```
这样就解决了节点以及命名空间的问题。这里是在RECEIVE，接收阶段处理了报文，且把报文放进了exchange中。
因为拦截器不同阶段的数据流不同，到输出拦截器的时候是读不到输入的报文的，所以要存起来，便于后面取出来记录日志。

(2) 获取Header信息：
```
package hls.core.ws.interceptor;

import hls.core.ws.utils.SmUtils;
import org.apache.cxf.binding.soap.SoapHeader;
import org.apache.cxf.binding.soap.SoapMessage;
import org.apache.cxf.binding.soap.interceptor.AbstractSoapInterceptor;
import org.apache.cxf.binding.soap.saaj.SAAJInInterceptor;
import org.apache.cxf.interceptor.Fault;
import org.apache.cxf.phase.Phase;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.w3c.dom.Element;
import org.w3c.dom.NodeList;
import org.apache.cxf.headers.Header;

import javax.xml.soap.SOAPException;
import java.io.IOException;
import java.io.InputStream;
import java.util.List;
import java.util.Properties;

/**
 * @Author: ever
 * @DATE: 2019-12-02 14:14
 * @Description 接口服务端流入拦截器(校验Header信息)
 */
public class ServiceInPreProtocolInterceptor extends AbstractSoapInterceptor {
    protected static Logger log = LoggerFactory.getLogger(ServiceInPreProtocolInterceptor.class);

    public ServiceInPreProtocolInterceptor() {
        super(Phase.PRE_PROTOCOL);
        getAfter().add(SAAJInInterceptor.class.getName());
    }

    private static String USER_NAME = "USER_NAME";
    private static String PASSWORD  = "PASSWORD";


    @Override
    public void handleMessage(SoapMessage soapMessage) throws Fault {
        // 校验头信息
        System.out.println("开始验证用户信息！");
        List<Header> headers = soapMessage.getHeaders();
        for (Header header : headers) {
            SoapHeader soapHeader = (SoapHeader) header;
            // 取出SOAP的Header元素
            Element element = (Element) soapHeader.getObject();
            log.info("ELEMENT =" + element.toString());
            // XMLUtils.printDOM(element);
            NodeList userIdNodes = element.getElementsByTagName("qwe:username");
            NodeList pwdNodes = element.getElementsByTagName("qwe:password");
            if (userIdNodes.item(0) == null || pwdNodes.item(0) == null) {
                SOAPException soapExc = new SOAPException("请在Header中提供完整的用户名、密码信息");
                throw new Fault(soapExc);
            }
            String userName = userIdNodes.item(0).getTextContent();
            String password = pwdNodes.item(0).getTextContent();
            log.info("############ 打印帐号信息 ##############");
            log.info(userIdNodes.item(0) + "=" + userName);
            log.info(pwdNodes.item(0) + "=" + password);
            log.info("############  ————————  ##############");
            // 用户名及密码使用SM3加密后的密文进行传输，此处判断源数据与加密数据
            if (!SmUtils.verify(USER_NAME, userName) || !SmUtils.verify(PASSWORD, password)) {
                //认证失败则抛出异常，停止继续操作
                SOAPException soapExc = new SOAPException("请提供正确的用户名和密码！");
                throw new Fault(soapExc);
            }
        }
    }
}
```
可以对请求用户进行用户名、密码的校验

#### 1.3. 输出阶段
阶段名称|阶段功能描述
--|:--:
SETUP|Any set up for the following phases(设置阶段)
(PRE/USER/POST)_LOGICAL|Processing of objects about to marshalled
PREPARE_SEND|Opening of the connection(消息发送准备阶段，在该阶段创建Connection)
PRE_STREAM|流准备阶段
PRE_PROTOCOL|Misc protocol actions(协议准备阶段)
WRITE|Writing of the protocol message, such as the SOAP Envelope.(写消息阶段)
MARSHAL|Marshalling of the objects
(USER/POST)_PROTOCOL|Processing of the protocol message
(USER/POST)_STREAM|Processing of the byte level message(字节处理阶段，在该阶段把消息转为字节)
SEND|消息发送

### 1.4. 输出拦截器
修改报文、记录日志：
```
package hls.core.ws.interceptor;

import hls.core.ws.dto.HlsCusWsRequests;
import hls.core.ws.mapper.HlsCusWsRequestsMapper;
import org.apache.cxf.helpers.IOUtils;
import org.apache.cxf.interceptor.Fault;
import org.apache.cxf.io.CachedOutputStream;
import org.apache.cxf.message.Exchange;
import org.apache.cxf.message.Message;
import org.apache.cxf.phase.AbstractPhaseInterceptor;
import org.apache.cxf.phase.Phase;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.nio.charset.StandardCharsets;
import java.util.Date;

/**
 * @Author: ever
 * @DATE: 2019-12-02 14:14
 * @Description: 接口服务端流出拦截器(修改报文, 记录日志)
 */
public class ServiceOutPreStreamInterceptor extends AbstractPhaseInterceptor<Message> {
    protected static Logger log = LoggerFactory.getLogger(ServiceOutPreStreamInterceptor.class);

    @Autowired
    private HlsCusWsRequestsMapper hlsCusWsRequestsMapper;

    public ServiceOutPreStreamInterceptor() {
        super(Phase.PRE_STREAM);
    }

    @Override
    public void handleMessage(Message message) throws Fault {
        try {
            OutputStream os = message.getContent(OutputStream.class);

            CachedStream cs = new CachedStream();

            message.setContent(OutputStream.class, cs);

            message.getInterceptorChain().doIntercept(message);

            CachedOutputStream csnew = (CachedOutputStream) message.getContent(OutputStream.class);
            InputStream in = csnew.getInputStream();

            String xml = IOUtils.toString(in);
            log.info("replaceBegin" + xml);
            Exchange exchange = message.getExchange();
            exchange.put("output.content.origin", xml);

            // 记录日志
            record(message);

            xml = xml.replace("return", "qwerty:response");//替换成你需要的格式
            log.info("replaceAfter" + xml);

            //这里对xml做处理，处理完后同理，写回流中
            IOUtils.copy(new ByteArrayInputStream(xml.getBytes(StandardCharsets.UTF_8)), os);

            cs.close();
            os.flush();

            message.setContent(OutputStream.class, os);
        } catch (Exception e) {
            log.error("输入流解析错误 : " + "\n" + e);
        }
    }

    /**
     * 记录日志
     */
    private void record(Message message) {
        Exchange exchange = message.getExchange();

        HlsCusWsRequests hlsCusWsRequests = new HlsCusWsRequests();
        hlsCusWsRequests.setRequestDate(new Date());
        hlsCusWsRequests.setParameterType("soap message");
        hlsCusWsRequests.setRequestJson(exchange.get("input.content.origin").toString() + "\n" 
            + exchange.get("input.content").toString());
        hlsCusWsRequests.setResponseJson(exchange.get("output.content.origin").toString());
        hlsCusWsRequests.setRequestWsdlUrl("{http://qwerty.com}FunctionnameService");
        hlsCusWsRequests.setFunctionName("FunctionName");
        hlsCusWsRequestsMapper.insertPaymentRequest(hlsCusWsRequests);
    }

    private class CachedStream extends CachedOutputStream {

        private CachedStream() {

            super();

        }

        protected void doFlush() throws IOException {

            currentStream.flush();

        }

        protected void doClose() throws IOException {

        }

        protected void onWrite() throws IOException {

        }

    }
}

```
从exchange中取出请求报文，记录日志。
## Z. 参考
[1. cxf拦截器，实现对接收到的报文和发送出去的报文格式自定义](https://blog.csdn.net/zhaofuqiangmycomm/article/details/78702125)
[2. CXF 入门：创建一个基于SOAPHeader的安全验证(CXF拦截器使用)](https://www.iteye.com/blog/jyao-1343722)
[3. CXF实战之拦截器Interceptor(四)](https://blog.csdn.net/accountwcx/article/details/47102319)
