

dubbo:service和dubbo:reference

mock：consumer端  reference中属性设置为true，当超时时间到后，调用本地提供的*Mock类进行处理

timeout: 以consumer端为准， dubbo:consumer 中timeou属性为主，没有该属性时，以reference中timeout属性为主，provider端中的timeout为辅

check:





服务远程运维命令：telnet ip host   如：telnet localhost 22222 查看dubbo服务，ls 查看列表

链接:

- [dubbo官网](https://dubbo.apache.org/zh-cn/docs/user/quick-start.html)
- [dubbo-admin](https://github.com/apache/dubbo-admin/tree/master)





JDK中的SPI：

