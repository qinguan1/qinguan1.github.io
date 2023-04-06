## Spring Security 的设计模式

作者：qinguan
<br/>博客：[https://bugstack.cn](https://bugstack.cn)

> 故不积跬步，无以至千里；不积小流，无以成江海！🌻


AuthenticationManager.authenticate
👇
ProviderManager.authenticate
⭕ :: supports -> provider 
👇
provider.authenticate

### 1. 模板方法模式

**简述模板方法：**
Template Pattern(模板方法模式)是一个抽象类公开定义了执行它的方法的模板。它的子类可以按需要**重写方法**实现，但调用将以抽象类中定义的方式进行，这是一种**行为型模式**。












参考博客：https://blog.csdn.net/u012702547/article/details/107682069

















