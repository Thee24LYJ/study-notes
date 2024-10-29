# C++设计模式(部分笔记)

参考笔记：[Design-Patterns-CPP](https://design-patterns-cplusplus.readthedocs.io/zh_CN/latest/)

### 12 单件模式

> 注意构造函数和拷贝构造函数需要设为 private

![image-20230912102224891](C:\Users\Administrator\Documents\typora-images\image-20230912102224891.png)

> 工厂模式绕开 new 解决 new 带来的紧耦合问题，单件模式保证只存在一个实例
>
> 双检查锁，产生一个实例，但由于内存读写 reorder 不安全，导致双检查锁失效
>
> reorder(指令集层面)：执行指令和理论上不同(编译器优化原因 )
>
> 双检查锁解决 reorder 的方法(现在也可直接使用 volatile)：
>
> ![image-20230912104058748](C:\Users\Administrator\Documents\typora-images\image-20230912104058748.png)

![image-20230912104316115](C:\Users\Administrator\Documents\typora-images\image-20230912104316115.png)

![image-20230912104343213](C:\Users\Administrator\Documents\typora-images\image-20230912104343213.png)

### 13 享元模式

![image-20230912104858354](C:\Users\Administrator\Documents\typora-images\image-20230912104858354.png)

![image-20230912104912251](C:\Users\Administrator\Documents\typora-images\image-20230912104912251.png)

![image-20230912105608998](C:\Users\Administrator\Documents\typora-images\image-20230912105608998.png)

### 14 门面模式

![image-20230913100825672](C:\Users\Administrator\Documents\typora-images\image-20230913100825672.png)

![image-20230913101403459](C:\Users\Administrator\Documents\typora-images\image-20230913101403459.png)

![image-20230913101856738](C:\Users\Administrator\Documents\typora-images\image-20230913101856738.png)

### 15 代理模式

![image-20230913102104652](C:\Users\Administrator\Documents\typora-images\image-20230913102104652.png)

![image-20230913104041259](C:\Users\Administrator\Documents\typora-images\image-20230913104041259.png)

### 16 适配器

![image-20230914100616532](C:\Users\Administrator\Documents\typora-images\image-20230914100616532.png)

![image-20230914102010801](C:\Users\Administrator\Documents\typora-images\image-20230914102010801.png)

### 17 中介者

![image-20230914102356317](C:\Users\Administrator\Documents\typora-images\image-20230914102356317.png)

![image-20230914104049916](C:\Users\Administrator\Documents\typora-images\image-20230914104049916.png)

# 18 状态模式

![image-20230921221321348](C:\Users\Administrator\Documents\typora-images\image-20230921221321348.png)

![image-20230921221428143](C:\Users\Administrator\Documents\typora-images\image-20230921221428143.png)

![image-20230921222642746](C:\Users\Administrator\Documents\typora-images\image-20230921222642746.png)

![image-20230921222923486](C:\Users\Administrator\Documents\typora-images\image-20230921222923486.png)

# 19 备忘录

![image-20230921223211706](C:\Users\Administrator\Documents\typora-images\image-20230921223211706.png)

![image-20230921223746790](C:\Users\Administrator\Documents\typora-images\image-20230921223746790.png)

![image-20230921224814201](C:\Users\Administrator\Documents\typora-images\image-20230921224814201.png)

# 20 组合模式

![image-20230922222804657](C:\Users\Administrator\Documents\typora-images\image-20230922222804657.png)

![image-20230922222858099](C:\Users\Administrator\Documents\typora-images\image-20230922222858099.png)

![image-20230922222943952](C:\Users\Administrator\Documents\typora-images\image-20230922222943952.png)

![image-20230922224217182](C:\Users\Administrator\Documents\typora-images\image-20230922224217182.png)

# 21 迭代器

![image-20230923220639674](C:\Users\Administrator\Documents\typora-images\image-20230923220639674.png)

![image-20230923220701197](C:\Users\Administrator\Documents\typora-images\image-20230923220701197.png)

![image-20230923221603122](C:\Users\Administrator\Documents\typora-images\image-20230923221603122.png)

# 22 职责链

![image-20230923221715010](C:\Users\Administrator\Documents\typora-images\image-20230923221715010.png)

![image-20230923221834023](C:\Users\Administrator\Documents\typora-images\image-20230923221834023.png)

![image-20230923222737526](C:\Users\Administrator\Documents\typora-images\image-20230923222737526.png)

# 23 命令模式

![image-20230924153821223](C:\Users\Administrator\Documents\typora-images\image-20230924153821223.png)

![image-20230924155313557](C:\Users\Administrator\Documents\typora-images\image-20230924155313557.png)

![image-20230924155205564](C:\Users\Administrator\Documents\typora-images\image-20230924155205564.png)

# 24 访问器

> double dispatch(两次多态辨析)
>
> 缺点：==必须确定==Element 的所有子类个数，因为 Visitor 中需要使用

![image-20230924155935775](C:\Users\Administrator\Documents\typora-images\image-20230924155935775.png)

![image-20230924161849383](C:\Users\Administrator\Documents\typora-images\image-20230924161849383.png)

![image-20230924161307285](C:\Users\Administrator\Documents\typora-images\image-20230924161307285.png)

![image-20230924161635197](C:\Users\Administrator\Documents\typora-images\image-20230924161635197.png)

# 25 解析器

![image-20230924162813693](C:\Users\Administrator\Documents\typora-images\image-20230924162813693.png)

![image-20230924162854401](C:\Users\Administrator\Documents\typora-images\image-20230924162854401.png)

![image-20230924164233659](C:\Users\Administrator\Documents\typora-images\image-20230924164233659.png)

![image-20230924164723037](C:\Users\Administrator\Documents\typora-images\image-20230924164723037.png)

# 总结

- 目标：管理变化，提高复用
- 两种手段：分解、抽象

- 八大原则

依赖倒置原则(DIP)
开放封闭原则(OCP)
单一职责原则(SRP)
Liskov 替换原则(LSP)
接口隔离原则(1SP)
对象组合优于类继承
封装变化点
面向接口编程

- 重构技法

静态->动态
早绑定->晚绑定
继承->组合
编译时依赖->运行时依赖
紧耦合->松耦合

![image-20230927200952602](C:\Users\Administrator\Documents\typora-images\image-20230927200952602.png)

所有模式基本上都是采用的==第三种==C++对象模型
