## java集合Map专题

作者：qinguan
<br/>博客：[https://bugstack.cn](https://bugstack.cn)

> 故不积跬步，无以至千里；不积小流，无以成江海！🌻




Spring 注解（AOP）不生效原因分析以及解决方案
原因：直接通过this去调用aop方法，则aop则不会生效
解决方案：获取对应的代理类，然后调用代理类的对应方法
