1. 初始化各种属性, 加载成对象
	- 读取环境属性(Enviroment)
	- 系统配置(spring.factories)
	- 参数(Arguments, application.properties)
2. 创建Spring容器对象ApplicationContext, 加载各种配置
3. 在容器创建前, 通过监听器机制, 应对不同阶段加载数据, 更新数据的需求
4. 容器初始化过程中追加各种功能, 例如统计时间, 输出日志等



