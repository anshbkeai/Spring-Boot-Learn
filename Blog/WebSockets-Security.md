**WEB-SOCKETS**
```
SOCKETS  ->  WEB  SOCKETS  (FULL  DUPLEX ,  Pub/sub  type ,  )
```
Client Subscibe  the  topic and if  any repsoen  on  he   topic  will be  Breoadcasted  to  the   Client  about  that   , that  is  Simple  way  that  it  works 
In  spring boot  ,  MessageBroker,  Stomp  endpoint  ,  
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig  implements  WebSocketMessageBrokerConfigurer{

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").withSockJS(); // WebSocket endpoint
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic"); // Enables in-memory broker
        registry.setApplicationDestinationPrefixes("/app");
    }

}
```
**The  Most  Loving part  is  to  just  to  visit  about  the  Annotation  and  leanr  about  the Class   that  thye  implement  and  made  about  the   things  ready  for  that  part** 
```
Lifecycle of WebSocket
Handshake => Starts with a regular HTTP request
If accepted, it upgrades to WebSocket (101 Switching Protocols)
Open Connection => Both sides can send/receive freely
Messaging => JSON/text/binary data
Close Connection => Either side can close it

Under the  hood  Still  usess abut  the   TCP  connection  
```
---
That  was  the   part about  understadning  of  the   web  sockets  
Now  improving  more  forward  to  the   security  part  about  the   Jwt  n the   simple  terms   
1.  Authication  =>  Normal about  the  Http  Auth we  have  in  here  . and   says  aboout  that 

