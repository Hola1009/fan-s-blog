MyBatisPlus通过扫描实体类，并基于反射获取实体类信息作为数据库表信息。

## 约定
- 类名驼峰转下划线作为表名
- 名为id的字段作为主键
- 变量名驼峰转下划线作为表的字段名

MybatisPlus中比较常用的几个注解如下:

- @TableName:用来指定表名
- @Tableld:用来指定表中的主键字段信息
- @TableField:用来指定表中的普通字段信息

IdType枚举:
- AUTO:数据库自增长
- INRUT:通过set方法自行输入
- ASSIGN ID:分配lD，接口ldentifierGenerator的方法nextld来生成id 默认实现类为DefaultldentifierGenerator雪花算法

使用@TableField的常见场景:
- 成员变量名与数据库字段名不一致
- 成员变量名以is开头，且是布尔值成员
- 变量名与数据库关键字冲突

![[Pasted image 20240528152639.png|left|300]]

## 常见配置
![[Pasted image 20240528153234.png]]



## 对于我的补充
### wrapper 的 select ()
里面的值是 就是 正常语句后面要跟的字段


### UpdateWrapper 的用法
.setSql("条件语句") 如 xxx = yyy 但是并不符合 规范, 业务代码 不要 有 sql 语句

有 LambdaQueryWrapper 同样也有  LambdaUpdateWrapper

### 将 MP 和 xml 文件 结合使用


1. 先在 业务层 用 wrapper 创建条件where 条件
2. 在 自定义 mapper 方法 的参数表中加入入参(需要加注解)
3. 自定义 sql 并使用 Wrapper 条件

![[Pasted image 20240528172401.png|left|500]]

## Service 接口

![[Pasted image 20240528181501.png|left|500]]


## 批量插入

mp 批处理 需要 开启 rewriteBatchedStatements=true 参数 拼接在配置文件的 url 后面即可

## 代码生成

下个插件
![[Pasted image 20240528192656.png|left|400]]
配一下
![[Pasted image 20240528192735.png]]
就可以直接生成代码了

## 静态工具

和 IService 差不多 就是要传 操作的实体类对象

这种静态工具是为了解决 业务层多表操作 循环依赖的的结果 service 类 彼此依赖

![[Pasted image 20240528225429.png]]

## 属性赋值
使用 BeanUtil.copyProperties(source, target.class)

## MP实现逻辑删除

MybatisPlus提供了逻辑删除功能，无需改变方法调用的方式，而是在底层帮我们自动修改CRUD的语句。我们要做的就是在application.yaml文件中配置逻辑删除的字段名称和值即可:

```java
mybatis-plus:  
  global-config:  
    db-config:  
      logic-delete-field: deleted  // 逻辑删除字段名
      logic-delete-value: 1  // 删除标识
      logic-not-delete-value: 0  // 不删除标识
```

查询的时候会追加这么一段
```mysql
AND deleted = 0
```

数据量会越来越大, 影响查询效率
sql语句需要对逻辑字段做判断, 影响查询效率

不想这样的话也可以用数据迁移

## 自动映射枚举

### 声明通用枚举属性

```java
public class User {

private String name; // 名字

private AgeEnum age; // 年龄

private GradeEnum grade; // 年级

}
```

#### 方式 一: 使用 @EnumValue 注解

```java
public enum GradeEnum {

	PRIMARY(1, "小学"),
	
	SECONDORY(2, "中学"),
	
	HIGH(3, "高中");
	
	GradeEnum(int code, String descp) {
	
		this.code = code;
		
		this.descp = descp;
	
	}
	
	@EnumValue // 标记数据库存的值是code
	private final int code;
	
	// 其他属性...

}
```


#### 方式二 : 枚举属性 实现 IIEnum 接口

```java
public enum AgeEnum implements IEnum<Integer> {
    ONE(1, "一岁"),
    TWO(2, "二岁"),
    THREE(3, "三岁");

    private int value;
    private String desc;

    @Override
    public Integer getValue() {
        return this.value;
    }
}
```

### 配置 MyBatis-Plus 自动映射枚举

```java
@Configuration
public class MybatisPlusAutoConfiguration {

    @Bean
    public MybatisPlusPropertiesCustomizer mybatisPlusPropertiesCustomizer() {
        return properties -> {
            GlobalConfig globalConfig = properties.getGlobalConfig();
            globalConfig.setBanner(false);
            MybatisConfiguration configuration = new MybatisConfiguration();
            configuration.setDefaultEnumTypeHandler(MybatisEnumTypeHandler.class);
            properties.setConfiguration(configuration);
        };
    }
}
```

### 序列化枚举值为前端返回值

枚举默认返回的值为 枚举项的名字

自定义需要返回的值 用 @JsonValue 注解

```java
@EnumValue
@JsonValue // 标记响应json值
private final int code;
```

### json 对象处理
![[Pasted image 20240529092445.png|left|600]]

## 插件
```java
@Configuration
@MapperScan("scan.your.mapper.package")
public class MybatisPlusConfig {

    /**
     * 添加分页插件
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL)); // 如果配置多个插件, 切记分页最后添加
        // 如果有多数据源可以不配具体类型, 否则都建议配上具体的 DbType
        return interceptor;
    }
}
```

## 通用分页实体
工具类
```java
@Data  
@NoArgsConstructor  
@AllArgsConstructor  
public class PageDTO<V> {  
    private Long total;  
    private Long pages;  
    private List<V> list;  
  
    /**  
     * 返回空分页结果  
     * @param p MybatisPlus的分页结果  
     * @param <V> 目标VO类型  
     * @param <P> 原始PO类型  
     * @return VO的分页对象  
     */  
    public static <V, P> PageDTO<V> empty(Page<P> p){  
        return new PageDTO<>(p.getTotal(), p.getPages(), Collections.emptyList());  
    }  
  
    /**  
     * 将MybatisPlus分页结果转为 VO分页结果  
     * @param p MybatisPlus的分页结果  
     * @param voClass 目标VO类型的字节码  
     * @param <V> 目标VO类型  
     * @param <P> 原始PO类型  
     * @return VO的分页对象  
     */  
    public static <V, P> PageDTO<V> of(Page<P> p, Class<V> voClass) {  
        // 1.非空校验  
        List<P> records = p.getRecords();  
        if (records == null || records.size() <= 0) {  
            // 无数据，返回空结果  
            return empty(p);  
        }  
        // 2.数据转换  
        List<V> vos = BeanUtil.copyToList(records, voClass);  
        // 3.封装返回  
        return new PageDTO<>(p.getTotal(), p.getPages(), vos);  
    }  
  
    /**  
     * 将MybatisPlus分页结果转为 VO分页结果，允许用户自定义PO到VO的转换方式  
     * @param p MybatisPlus的分页结果  
     * @param convertor PO到VO的转换函数  
     * @param <V> 目标VO类型  
     * @param <P> 原始PO类型  
     * @return VO的分页对象  
     */  
    public static <V, P> PageDTO<V> of(Page<P> p, Function<P, V> convertor) {  
        // 1.非空校验  
        List<P> records = p.getRecords();  
        if (records == null || records.size() <= 0) {  
            // 无数据，返回空结果  
            return empty(p);  
        }  
        // 2.数据转换  
        List<V> vos = records.stream().map(convertor).collect(Collectors.toList());  
        // 3.封装返回  
        return new PageDTO<>(p.getTotal(), p.getPages(), vos);  
    }  
}
```


这个类可以用来被其他类 继承, 进行查询的时候, 先通过 toMapPage 创建 page 对象, 然后 调用 IService 的 page 方法 传入 page 对象 和 wrapper 对象 来进行查询
```java
@Data  
public class PageQuery {
	// 当前页数 和 页码
    private Integer pageNo;  
    private Integer pageSize;
    // 排序 的 字段 和 升序或降序  
    private String sortBy;
    private Boolean isAsc;  
  
    public <T>  Page<T> toMpPage(OrderItem ... orders){  
        // 1.分页条件  
        Page<T> p = Page.of(pageNo, pageSize);  
        // 2.排序条件  
        // 2.1.先看前端有没有传排序字段  
        if (sortBy != null) {  
            p.addOrder(new OrderItem(sortBy, isAsc));  
            return p;  
        }  
        // 2.2.再看有没有手动指定排序字段  
        if(orders != null){  
            p.addOrder(orders);  
        }  
        return p;  
    }  
	
    public <T> Page<T> toMpPage(String defaultSortBy, boolean isAsc){  
        return this.toMpPage(new OrderItem(defaultSortBy, isAsc));  
    }  
  
    public <T> Page<T> toMpPageDefaultSortByCreateTimeDesc() {  
        return toMpPage("create_time", false);  
    }  
  
    public <T> Page<T> toMpPageDefaultSortByUpdateTimeDesc() {  
        return toMpPage("update_time", false);  
    }  
}
```

