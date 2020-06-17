# Interview-Best-Practices

1. session 工作原理?
浏览器第一次请求网站， 服务端生成 Session ID。
把生成的 Session ID 保存到服务端存储中。
把生成的 Session ID 返回给浏览器，通过 set-cookie。
浏览器收到 Session ID， 在下一次发送请求时就会带上这个 Session ID。
服务端收到浏览器发来的 Session ID，从 Session 存储中找到用户状态数据，会话建立。
此后的请求都会交换这个 Session ID，进行有状态的会话。
