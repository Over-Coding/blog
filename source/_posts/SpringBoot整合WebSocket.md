---
title: SpringBoot整合WebSocket
date: 2020-01-02 14:25:14
categories:
	- SpringBoot
tags: 
	- SpringBoot
	- WebSocket
toc: true
---



### 1、引入jar包
```xml
<dependency>    
    <groupId>org.springframework.boot</groupId>    
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```



### 2、自定义HandshakeInterceptor

```java
@Component
public class WebSocketHandshakeInterceptor implements HandshakeInterceptor {   

    @Override    
    public boolean beforeHandshake(ServerHttpRequest serverHttpRequest, 
        ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Map<String, Object> attributes)       throws Exception { 
        
        if (serverHttpRequest instanceof ServletServerHttpRequest) {            
            //强转请求类型            
            HttpServletRequest servletRequest = ((ServletServerHttpRequest) serverHttpRequest).getServletRequest();            
            //通过请求参数获取socketId,用于作为socket账号信息            
            String socketId = servletRequest.getParameter("socket_id");            
            if(StringUtils.isEmpty(socketId)){                
                return false;           
            }            
            attributes.put("socket_id", socketId);       
        }        
        return true;   
    }   
    
    @Override    
    public void afterHandshake(ServerHttpRequest serverHttpRequest, 
         ServerHttpResponse serverHttpResponse,WebSocketHandler webSocketHandler, Exception e) {   
    }
    
}
```



### 3、自定义WebSocketHandler

```java
@Component
public class WebSocketHandler implements WebSocketHandler {

    //存储websocket session 集合
    public static ConcurrentHashMap<String, WebSocketSession> webSocketSessionMap = new ConcurrentHashMap<>();

    /**
     * 建立连接之后
     * @param webSocketSession
     * @throws Exception
     */
    @Override
    public void afterConnectionEstablished(WebSocketSession webSocketSession) throws Exception {
        //获取socketId
        String websocket_id = (String) webSocketSession.getAttributes().get("socket_id");
        //存储socketId和session对象
        webSocketSessionMap.put(websocket_id,webSocketSession);
        LogUtil.debug(websocket_id + ",建立连接成功...");
    }

    /** 
    * 获取到消息 
    * @param webSocketSession 
    * @param webSocketMessage 
    * @throws Exception 
    */
    @Override
    public void handleMessage(WebSocketSession webSocketSession, WebSocketMessage<?> webSocketMessage) throws                                               Exception {    
    
        LogUtil.debug("获取发来的消息," + webSocketMessage.getPayload().toString());
    }

    /** 
    * 连接错误 
    * @param webSocketSession 
    * @param throwable 
    * @throws Exception 
    */
    @Override
    public void handleTransportError(WebSocketSession webSocketSession, Throwable throwable) throws Exception {    
        //获取socketId    
        String websocket_id = (String) webSocketSession.getAttributes().get("socket_id");   
        //根据socketId删除session对象    
        webSocketSessionMap.remove(websocket_id);    
        LogUtil.debug(websocket_id + ",连接出现错误...");
    }

    /** 
    * 关闭连接 
    * @param webSocketSession 
    * @param closeStatus 
    * @throws Exception 
    */
    @Override
    public void afterConnectionClosed(WebSocketSession webSocketSession, CloseStatus closeStatus) throws Exception {    
        //获取socketId    
        String websocket_id = (String) webSocketSession.getAttributes().get("socket_id");    
        //根据socketId删除session对象    
        webSocketSessionMap.remove(websocket_id);   
        LogUtil.debug(websocket_id + ",关闭连接...");
    }
    
    @Override
    public boolean supportsPartialMessages() {    
        return false;
    }

    /** 
    * 给所有的用户发送信息 
    * @param message 信息 
    */
    public void sendMessageToAllUser(String message){    
         //遍历session集合    
        ConcurrentHashMap.KeySetView<String, WebSocketSession> keySet = webSocketSessionMap.keySet();   
        Iterator<String> iterator = keySet.iterator();    
        while (iterator.hasNext()){       
            //获取socketId        
            String websocket_id = iterator.next();        
            //获取session对象        
            WebSocketSession webSocketSession = webSocketSessionMap.get(websocket_id);       
             if(webSocketSession != null && webSocketSession.isOpen()){            
                try {                
                     //发送消息                
                    webSocketSession.sendMessage(new TextMessage(message));           
                } catch (IOException e) {                
                    LogUtil.error("websocket 发送数据失败，失败原因：" + e);           
                }       
             }    
        }
    }

    /** 
    * 给单个用户发送信息 
    * @param socketId socket账户 
    * @param message 消息 
    */
    public void sendMessageToOneUser(String socketId,String message){
        if(!StringUtils.isEmpty(socketId)){        
             //判断session集合中存在socketId        
            if(webSocketSessionMap.containsKey(socketId)){           
             //获取session对象            
            WebSocketSession webSocketSession = webSocketSessionMap.get(socketId);            
                  if(webSocketSession != null && webSocketSession.isOpen()){                
                        try {                   
                             //发送消息                    
                            webSocketSession.sendMessage(new TextMessage(message));               
                        } catch (IOException e) {                   
                            LogUtil.error("websocket 发送数据失败，失败原因：" + e);               
                        }            
                    }       
             }    
        }
    }
}
```



### 4、注册webSocket组件

* 实现 WebSocketConfigurer 接口，重写 registerWebSocketHandlers 方法，这是一个核心实现方法，配置 websocket 入口，允许访问的域、注册 Handler、SockJs 支持和拦截器。

* registry.addHandler()注册和路由的功能，当客户端发起 websocket 连接，把 /path 交给对应的 handler 处理，而不实现具体的业务逻辑，可以理解为收集和任务分发中心。

* addInterceptors，顾名思义就是为 handler 添加拦截器，可以在调用 handler 前后加入我们自己的逻辑代码。

* setAllowedOrigins(String[] domains),允许指定的域名或 IP (含端口号)建立长连接，如果只允许自家域名访问，这里轻松设置。如果不限时使用”* ”号，如果指定了域名，则必须要以 http 或 https 开头。

```java
@Configuration
public class WebSocketConfig implements WebSocketConfigurer {    

    //拦截器    
    @Autowired    
    private webSocketHandshakeInterceptor handshakeInterceptor;    
    //websocket控件    
    @Autowired    
    private sebSocketHandler webSocketHandler;    

    @Override    
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {        
             //部分 支持websocket 的访问链接,允许跨域        
            registry.addHandler(webSocketHandler,"/websocket")
                        .addInterceptors(handshakeInterceptor).setAllowedOrigins("*");        
            //部分 不支持websocket的访问链接,允许跨域       
           registry.addHandler(webSocketHandler,"/sockjs/websocket")
                        .addInterceptors(handshakeInterceptor).setAllowedOrigins("*").withSockJS();
     }
}
```



### 5、开启WebSocket

一定不要忘添加@EnableWebSocket

```java
@EnableWebSocket
@SpringBootApplication
public class WebSocketApplication {    
    public static void main(String[] args) {        
        SpringApplication.run(WebSocketApplication.class, args);    
    }
}
```