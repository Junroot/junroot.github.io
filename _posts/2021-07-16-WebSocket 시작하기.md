---
title:  "WebSocket 시작하기"
categories: programming
tags: HTTP Spring
---

## 클라이언트가 서버에 갱신된 정보를 어떻게 빠르게 가져올 수 있을까?

### 이런 고민을 한 이유: HTTP의 한계
    - 무상태성: 클라이언트의 상태를 기억하지 않기 때문에 클라이언트에게 갱신된 정보를 알려줄 수 없다.
    - 비연결성: 1개의 리퀘스트로 부터 응답을 하면 연결이 끊어진다.
### Polling
주기적으로 클라이언트가 request를 보내 새로운 정보를 갱신하는 방법

- 장점
    - 구현이 단순하다.
- 단점
    - 주기적으로 HTTP 연결을 맺고 끊는게 상당한 클라이언트와 서버 측에서 모두 큰 부담이 된다.  이를 어느 정도 해결하기 위해 전송하는 데이터 양을 줄이기 위해 ajax를 사용한다.
- Long Polling(HTTP/1.1): 클라이언트와 서버가 계속 연결을 맺고 끊는 것을 줄이기 위해서 만든 방법. 일단 클라이언트가 서버로 HTTP 요청을 보낸 후, 그 상태로 계속 기다린다. 서버에서 해당 클라이언트로 전달할 정보가 생기면 그 순간 응답 메시지를 전달한 후 연결이 해제된다.
된다.
- 장점
    - Polling 방식보다 어느 정도 클라이언트와 서버의 부담이 줄었다.
- 단점
    - 정보 갱신이 잦으면 Polling 방식과 별 차이가 없다.
    - 정보 갱신이 발생하여 클라이언트에게 알려주면 클라이언트에서 다시 request를 동시에 보내기 때문에 순간적으로 서버에 부담이 증가한다.
### HTTP Streaming(HTTP/1.1)
클라이언트가 HTTP 요청을 서버에게 보내면 연결을 해제하지 않은 상태에서 서버는 응답을 계속 보낸다.
- 장점
    - 연결을 맺고 끊는 것에 발생하는 부담을 해결했다.
- 단점
    - 하나의 TCP 포트를 이용해서 동시에 읽고 쓰기를 할 수 없기 때문에, 서버에서 클라이언트로 메시지를 보낼 수 있으나 동시에 클라이언트에서 서버로 메시지를 보내는 것은 어렵다. (추가적인 학습 필요)

장시간 HTTP 요청을 대기하더라도 서버가 새로운 정보가 있을 때 클라이언트에 보내는 웹 애플리케이션 모델을 Comet이라고 부른다. Long Polling과 HTTP Streaming이 Comet에 해당한다. HTTP/1.1에서 하나의 TCP 연결에서 여러개의 HTTP 메시지가 주고받는 것이 구현되었다.

### WebSocket
HTTP의 근본적인 문제(무상태성, 비연결성)를 해결하기 위해 만들어진 프로토콜. 하지만 HTTP request 메시지를 변형없이 사용할 수 있기 때문에 80번과 433번 포트에서 동작할 수 있다. 웹소켓은  연결을 맺는 과정에서 일반 TCP 소켓과 다르게 HTTP 프로토콜을 사용한다. 따라서 추가적인 방화벽 설정을 할 필요가 없다.
- 장점
      - 클라이언트의 상태를 기억할 수 있다.
      - 연결이 유지된다.
      - 일반 HTTP 통신과정에서 발생하는 HTTP와 TCP 연결에 필요한 비용을 줄일 수 있다.
      - 불필요한 헤더가 사라지기 때문에 통신량을 줄일 수 있다.
      - 웹 표준이다.
- 단점
    - 오래된 브라우저에서 지원하지 않는다.
    - 서버와 클라이언트 간의 소켓 연결 자체가 자원을 소비하기 때문에 트래픽이 많은 서버 같은 경우 CPU에 부담이 될 수 있다.
    - 클라이언트의 상태를 기억하는 추가적인 구현이 필요하다. (자동 재연결, 연결 종료)
  
### SSE(Server-Sent Events)
HTTP Steaming 중 하나. HTTP를 통해 초기 연결이 되면 서버가 클라이언트로 전송을 할 수 있는 기술. W3C에 의해 HTML5 일부로 표준화 되었다. 하지만 SSE는 서버에서 클라이언트로 밖에 전송하지 못한다. 단방향 채널이다.
- 장점
    - 전통적인 HTTP를 통해 전송하기 때문에 특별한 프로토콜이나 서버 구현이 필요하지 않다.
    - 웹 소켓에비해 해야될 선행작업이 적다. (재연결 내장 지원)
- 단점
    - 양방향 연결이 아니다. 즉, 클라이언트에서 서버로 보내는 방법은 제공하지 않는다.
    - 최대로 열릴 수 있는 연결 수가 브라우저 당 6개로 제한되어있다.
    - HTTP2.0과 함께 쓰면 웹소켓과 비슷한 성능을 보이는데, HTTP2.0은 상대적으로 지원하는 브라우저가 적다.
  
### WebTransport
HTTP/3 기반에서 동작하는 웹소켓 프로토콜. 아직 HTTP/3는 draft 단계이므로 생략한다.

## 웹 소켓의 이해

- 소켓 연결을 유지하기 때문에 양방향 통신이 가능하다. 즉, 웹에서 사용하는 소켓이라 할 수 있다.
- 웹소켓과 HTTP 프로토콜 모두 OSI 모델에서 제7계층인 Application layer에 해당하며 제4계층인 TCP에 의존한다.
- TCP 소켓에서는 바이트 스트림을 사용하지만 웹 소켓에서는 UTF-8 포맷의 메시지 스트림만 허용한다.
  
## 웹소켓 통신 과정
### 1. WebSocket Handshake
1. 클라이언트 핸드쉐이크 요청

    ```json
    GET /chat HTTP/1.1
    Host: example.com:8000
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
    Sec-WebSocket-Version: 13
    ```

    - `Upgrade`: 웹소켓 프로토콜로 프로토콜을 변경한다는 요청
    - `Sec-WebSocket-Key`: 핸드쉐이크에 필요한 키
    - `Connection`: 프록시에 더 이상 전송하지 않는 헤더 필드를 지정하는 헤더다. `Upgarde` 헤더는 hop-by-hop 헤더기 때문에 `Connection:Upgrade` 가 필요하다.
2. 서버가 보내는 핸드쉐이크 응답

    ```jsx
    HTTP/1.1 101 Switching Protocols
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
    ```

    - `Sec-WebSocket-Accept`: `Sec-WebSocket-Key` 에 의해 생선된 값

웹소켓 서버는 이미 연결된 클라이언트들의 반복적인 연결(DoS 공격 등)을 막기위해 클라이언트의 소캣 상태를 추적해야된다. 

### 2. 데이터 프레임 교환
- [링크](https://developer.mozilla.org/ko/docs/Web/API/WebSockets_API/Writing_WebSocket_servers#%EB%8D%B0%EC%9D%B4%ED%84%B0_%ED%94%84%EB%A0%88%EC%9E%84_%ED%8F%AC%EB%A9%A7)를 참고하면 데이터 프레임의 구조를 알 수 있는데 직접 사용할 일이 있을지는 모르겠다.
- 핸드쉐이크가 끝난 이후 서버나 클라이언트는 언제나 ping을 보낼 수 있다. 수산지는 가능한 빨리 응답으로 pong 패킷을 보내야된다. 서버는 주적으로 ping을 보내 클라이언트가 살아있는 상태인지 체크할 수 있다.
### 3. 연결 닫기
1. 클라이언트가 close 프레임을 서버에게 보낸다.
2. 수신한 서버는 close 프레임을 보낸다.

![image](https://user-images.githubusercontent.com/4648244/125907860-e0b8a98a-33a6-420f-b93a-35838e044556.png)


## Spring으로 WebSocket 시작하기(STOMP)

### Maven 의존성 추가

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-websocket</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-messaging</artifactId>
    <version>5.2.2.RELEASE</version>
</dependency>
```

최신 버전은 [여기서](https://mvnrepository.com/)

또한 JSON 사용하여 메시지를 전달 하려면 Jackson 의존을 추가해야된다. Spring Boot의 경우에는 추가할 필요 없을 것으루 추정된다.

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.10.2</version>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId> 
    <version>2.10.2</version>
</dependency>
```

### Spring에서 WebSocket 활성화

`AbstractWebSocketMessageBrokerConfigurer` 클래스를 상속하고 `@EnableWebSocketMessageBroker` 어노테이션을 붙여준다.

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
         registry.addEndpoint("/chat");
         registry.addEndpoint("/chat").withSockJS();
    }
}
```

- `enableSimpleBroker()`: 클라이언트가 서버로부터 메시지를 받기위한 URL의 prefix 설정
- `setApplicationDestinationPrefixes()`: 클라이언트가 서버에게 메시지를 보내기 위한 URL의 prefix 설정
- `addEndpoint()`: 클라이언트와 서버가 connection(handshake)을 만들기 위한 endpoint 설정. `/app`으로 시작하는 URL의 요청은 `@Controller` 클래스의 `@MessageMapping` 메소드로 전달된다.
- `withSocketJS()`: SockJS fallback options 을 가능하게 해주는 설정.

fallback이란: 어떤 기능이 제대로 동작하지 않을 경우, 대처해주는 기능
STOMP란: WebSocket은 양방향 통신 프로토콜일 뿐, 메시지의 형식과 규칙을 정의하지는 않는다. STOMP는 이 메시지 형식과 규칙을 정의하는 프로토콜이다.

### 메시지 DTO 만들기

```java
public class Message {

    private String from;
    private String text;

    // getters and setters
}
```

### 메시지 컨트롤러 만들기

```java
@MessageMapping("/chat")
@SendTo("/topic/messages")
public OutputMessage send(Message message) throws Exception {
    String time = new SimpleDateFormat("HH:mm").format(new Date());
    return new OutputMessage(message.getFrom(), message.getText(), time);
}
```

- `@MessageMapping("/chat")`: connection을 위한 엔드포인트 지정
- `@SendTo("/topic/messages")`: 메시지를 보낼 URL 지정

### 클라이언트 사용 예시

```html
<html>
    <head>
        <title>Chat WebSocket</title>
        <script src="resources/js/sockjs-0.3.4.js"></script>
        <script src="resources/js/stomp.js"></script>
        <script type="text/javascript">
            var stompClient = null;
            
            function setConnected(connected) {
                document.getElementById('connect').disabled = connected;
                document.getElementById('disconnect').disabled = !connected;
                document.getElementById('conversationDiv').style.visibility 
                  = connected ? 'visible' : 'hidden';
                document.getElementById('response').innerHTML = '';
            }
            
            function connect() {
                var socket = new SockJS('/chat');
                stompClient = Stomp.over(socket);  
                stompClient.connect({}, function(frame) {
                    setConnected(true);
                    console.log('Connected: ' + frame);
                    stompClient.subscribe('/topic/messages', function(messageOutput) {
                        showMessageOutput(JSON.parse(messageOutput.body));
                    });
                });
            }
            
            function disconnect() {
                if(stompClient != null) {
                    stompClient.disconnect();
                }
                setConnected(false);
                console.log("Disconnected");
            }
            
            function sendMessage() {
                var from = document.getElementById('from').value;
                var text = document.getElementById('text').value;
                stompClient.send("/app/chat", {}, 
                  JSON.stringify({'from':from, 'text':text}));
            }
            
            function showMessageOutput(messageOutput) {
                var response = document.getElementById('response');
                var p = document.createElement('p');
                p.style.wordWrap = 'break-word';
                p.appendChild(document.createTextNode(messageOutput.from + ": " 
                  + messageOutput.text + " (" + messageOutput.time + ")"));
                response.appendChild(p);
            }
        </script>
    </head>
    <body onload="disconnect()">
        <div>
            <div>
                <input type="text" id="from" placeholder="Choose a nickname"/>
            </div>
            <br />
            <div>
                <button id="connect" onclick="connect();">Connect</button>
                <button id="disconnect" disabled="disabled" onclick="disconnect();">
                    Disconnect
                </button>
            </div>
            <br />
            <div id="conversationDiv">
                <input type="text" id="text" placeholder="Write a message..."/>
                <button id="sendMessage" onclick="sendMessage();">Send</button>
                <p id="response"></p>
            </div>
        </div>

    </body>
</html>
```

```java
public class WebSockChatHandler extends TextWebSocketHandler {

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String payload = message.getPayload();
        log.info("payload {}", payload);
        TextMessage textMessage = new TextMessage("Welcome chatting sever~^^");
        session.sendMessage(textMessage);
    }
}
```

### Websocket Config 작성

위에서 작성한 Websocket Handler를 URL 매핑을 하기위해서는 `WebSocketConfig` 를 구현하면된다.

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(webSocketHandler(), "/ws/chat").setAllowedOrigins("*");
    }

    @Bean
    public WebSocketHandler webSocketHandler(){
        return new WebSocketHandler();
    }
}
```

STOMP를 사용하는 것은 `WebSocketHandler`를 간접적으로 사용하는 것이다.

## STOMP가 더 좋다고 생각하는 개인적인 이유

- session을 직접 관리할 필요가 없다.
- 반환값을 사용자가 만든 DTO를 사용할 수 있다. 알아서 직렬화, 역직렬화 해준다.
- 메시지를 라우팅할 경로 쉽게 지정할 수 있다. ([https://stackoverflow.com/questions/42901062/spring-websockets-without-stomp-and-sockjs-but-with-message-broker-and-routing-s](https://stackoverflow.com/questions/42901062/spring-websockets-without-stomp-and-sockjs-but-with-message-broker-and-routing-s))

## 참고 자료

- [WebSocket과 Socket.io](https://d2.naver.com/helloworld/1336)
  
- [https://www.itfind.or.kr/publication/regular/weeklytrend/weekly/view.do?boardParam1=8034&boardParam2=8034](https://www.itfind.or.kr/publication/regular/weeklytrend/weekly/view.do?boardParam1=8034&boardParam2=8034)
  
- [https://ko.wikipedia.org/wiki/코멧_(프로그래밍)#cite_note-MASH-1](https://ko.wikipedia.org/wiki/%EC%BD%94%EB%A9%A7_(%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)#cite_note-MASH-1)
  
- [https://choseongho93.tistory.com/266](https://choseongho93.tistory.com/266)
  
- [https://hamait.tistory.com/792](https://hamait.tistory.com/792)
  
- [https://stackoverflow.com/questions/5195452/websockets-vs-server-sent-events-eventsource](https://stackoverflow.com/questions/5195452/websockets-vs-server-sent-events-eventsource)
  
- [https://stackoverflow.com/questions/12555043/my-understanding-of-http-polling-long-polling-http-streaming-and-websockets](https://stackoverflow.com/questions/12555043/my-understanding-of-http-polling-long-polling-http-streaming-and-websockets)
  
- [https://www.baeldung.com/websockets-spring](https://www.baeldung.com/websockets-spring)

- [https://www.section.io/engineering-education/getting-started-with-spring-websockets/](https://www.section.io/engineering-education/getting-started-with-spring-websockets/)

- [https://ratseno.tistory.com/71](https://ratseno.tistory.com/71)

- [https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp)

- [https://daddyprogrammer.org/post/4077/spring-websocket-chatting/](https://daddyprogrammer.org/post/4077/spring-websocket-chatting/)

- [https://velog.io/@hanblueblue/번역-Spring-4-Spring-WebSocket](https://velog.io/@hanblueblue/%EB%B2%88%EC%97%AD-Spring-4-Spring-WebSocket)