# gc

作者：qinguan
<br/>博客：[https://bugstack.cn](https://bugstack.cn)

> 故不积跬步，无以至千里；不积小流，无以成江海！🌻

## 如何识别垃圾对象
### 引用计数法
引用计数法是一种内存管理技术，它的基本思想是跟踪每个对象被引用的次数，并在引用计数为零时自动释放对象所占用的内存空间。
无法解决循环引用问题
![引用计数法](https://raw.githubusercontent.com/qinguan1/qinguan1.github.io/main/docs/assets/img/qinguan/引用计数法.png)

### 可达性分析算法

可达性分析算法是一种垃圾回收算法，用于判断一个对象是否可达（即是否还有引用指向它），从而确定是否可以将该对象回收。

可达性分析算法的基本思想是从一组根对象（通常称为GC root）开始，递归遍历所有可能被引用的对象，将所有遍历到的对象标记为“已访问”，最终未被标记的对象即为垃圾对象，可以被回收。这种算法也是目前主流垃圾回收器所使用的算法。

![可达性分析算法](https://raw.githubusercontent.com/qinguan1/qinguan1.github.io/main/docs/assets/img/qinguan/可达性分析算法.png)

## 有哪些垃圾回收算法

## 有哪些垃圾收集器



