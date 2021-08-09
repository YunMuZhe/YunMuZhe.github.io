## Bean创建的生命周期：

class(UserService.class) --->推断构造方法 --->实例化---> 对象---> 属性填充(依赖注入) ---> [初始化afterPropertiesSet(), 需要实现接口InitializeBean]---> [AOP ---> 代理对象---> 属性填充]---> Bean对象

一般来说，上面实例化后的对象和最终拿到的Bean对象是同一个对象，但AOP除外。

spring的bean生命周期

循环依赖如何解决

spring的代理模式

springboot脚手架启动类中的三大注解如何加载bean到容器中

手写一个starter需要注意什么

## 什么是AOP？

与OOP对比，AOP能处理一些横切性问题，这些横切性问题不会影响到主逻辑的实现，，但是会散落到代码的各个部分，难以维护。AOP就是把这些问题和主业务逻辑分开，达到与主业务逻辑解耦的目的。

传统OOP是从上到下的开发。

# springcloud中组件以及原理

