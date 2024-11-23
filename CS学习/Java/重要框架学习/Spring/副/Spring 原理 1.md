### 获取Bean的途径
1. 获得启动类.Run()方法返回的ApplicationContext实例, 调用context的getBean(xxx.class)方法
2. 通过注入获得

### Bean的作用域
1. **singleton**：单例作⽤域
2. **prototype**：原型作⽤域（多例作⽤域）
3. **request**：请求作⽤域
4. **session**：会话作⽤域  
5. **Application**: 全局作⽤域  每个ServletContext⽣命周期内, 创建新的实例

### Bean的生命周期
1. 先执⾏构造函数
2. 设置属性
3. Bean初始化
4. 使⽤Bean
5. 销毁Bean

### 第三方Bean管理
1. 要在启动类上加 `@ComponentScan({"路径"})` 注解
2. 或加 `@import(类对象)` 注解
3. 或实例化 `ImportSelector` 接口 在把要扫描的Bean路径放在一个字符串数组里即可                                                       再在启动类上加 加 `@import(实例化的接口的类对象)` 注解
4. 或创建一个注解, 把刚刚实例化接口的对象后的注解 `@import(实例化的类对象)` 放在其上, 再将这个注解放在启动类上
5. 或者让第三方自己实现注解, 把需要spring管理的路径放在其上
