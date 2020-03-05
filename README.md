# SpringBoot 使用WebSocket打造在线聊天室（基于注解）
## 一、打造 WebSocket 聊天客户端
### 使用步骤：1、获取WebSocket客户端对象。
	var webSocket = new WebSocket(url);    

### 使用步骤：2、获取WebSocket回调函数。
	webSocket.onmessage = function (event) {console.log('WebSocket收到消息：' + event.data);
| 事件类型 | WebSocket回调函数 | 事件描述 |
| open | webSocket.onopen | 当打开连接后触发 |
| message | webSocket.onmessage | 当客户端接收服务端数据时触发 |
| error | webSocket.onerror | 当通信异常时触发 |
| close | webSocket.onclose | 当连接关闭时触发 |	

### 使用步骤：3、发送消息给服务端
	webSokcet.send(jsonStr) 结合实际场景 本案例采用JSON字符串进行消息通信。

## 二、打造 WebSocket 聊天服务端
	首先在POM文件引入spring-boot-starter-websocket 、thymeleaf 、FastJson等依赖。	
### 使用步骤：1、开启WebSocket服务端的自动注册。	
	ServerEndpointExporter 是由Spring官方提供的标准实现，用于扫描ServerEndpointConfig配置类和@ServerEndpoint注解实例。使用规则也很简单：1.如果使用默认的嵌入式容器 比如Tomcat 则必须手工在上下文提供ServerEndpointExporter。2. 如果使用外部容器部署war包，则不要提供提供ServerEndpointExporter，因为此时SpringBoot默认将扫描服务端的行为交给外部容器处理。	
	
	@Configuration
    public class WebSocketConfig {
        @Bean
        public ServerEndpointExporter serverEndpointExporter() {
    
            return new ServerEndpointExporter();
        }
    }
### 使用步骤：2、创建WebSocket服务端。	
	核心思路：

	① 通过注解@ServerEndpoint来声明实例化WebSocket服务端。
	② 通过注解@OnOpen、@OnMessage、@OnClose、@OnError 来声明回调函数。
	
	    | 事件类型 |	WebSocket服务端注解 | 事件描述 |
        | open | @OnOpen | 当打开连接后触发 |
        | message |	@OnMessage | 当接收客户端信息时触发 |
        | error | @OnError | 当通信异常时触发 |
        | close | @OnClose | 当连接关闭时触发 |
    
    ③ 通过ConcurrentHashMap保存全部在线会话对象。 
     
        /**
         * 全部在线会话  PS: 基于场景考虑 这里使用线程安全的Map存储会话对象。
         */
        private static Map<String, Session> onlineSessions = new ConcurrentHashMap<>();
        
        
    ④ 通过会话对象 javax.websocket.Session 来发消息给客户端。
        /**
         * WebSocket 聊天消息类
         */
        package com.hehe.chat;
        
        import com.alibaba.fastjson.JSON;
        
        /**
         * WebSocket 聊天消息类
         */
        public class Message {
        
            public static final String ENTER = "ENTER";
            public static final String SPEAK = "SPEAK";
            public static final String QUIT = "QUIT";
        
            private String type;//消息类型
        
            private String username; //发送人
        
            private String msg; //发送消息
        
            private int onlineCount; //在线用户数
        
            public static String jsonStr(String type, String username, String msg, int onlineTotal) {
                return JSON.toJSONString(new Message(type, username, msg, onlineTotal));
            }
        
            public Message(String type, String username, String msg, int onlineCount) {
                this.type = type;
                this.username = username;
                this.msg = msg;
                this.onlineCount = onlineCount;
            }
        
            //这里省略get/set方法 请自行补充
        }
