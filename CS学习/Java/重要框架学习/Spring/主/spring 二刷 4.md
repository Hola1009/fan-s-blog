### Spring 其他配置标签
Spring 的 xml 标签大体上分为两类, 一种时默认标签, 一种是自定义标签
- 默认标签:就是不用额外导入其他命名空间约束的标签，例如\<bean>标签
- 自定义标签:就是需要额外引入其他命名空间约束，并通过前缀引用的标签

#### 默认标签

| 标签        | 作用                          |
| --------- | --------------------------- |
| \<beans>  | 一般作为 xml配置根标签，其他标签都是该标签的子标签 |
| \<bean>   | Bean的配置标签，上面已经详解了，此处不再阐述    |
| \<import> | 引入其他配置文件                    |
| \<alias\> | 指定Bean的别名标签，使用较少            |
