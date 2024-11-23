## mybatis plus 设置日志打印
```yml
mybatis-plus:  
  configuration:  
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```


## mybatis xml 的 in 语句
```java
<update id="decreaseArticlesTotal" parameterType="java.util.List">  
  update t_tag  
  set articles_total = articles_total - 1  where id in  
  <foreach collection="tagIds" item="id" index="index" open="(" close=")" separator=",">  
    #{id}  
  </foreach>  
</update>
```

注意这个foreach 怎么写的就好了

## Mybatis 实现多表查询
### 一对一
如, 一个订单对应一个用户, 可以通过 `association` 标签来解决
1. 定义实体类
```java
public class Order {    
	private int id;    
	private Date ordertime;    
	private double total;    
	//代表当前订单从属于哪一个客户    
	private User user;
	// 省略get set方法
}
```

```java
public class User {
   private int id;    
   private String username;    
   private String password;    
   private Date birthday;
   // 省略get set方法
}
```

2. 查询语句
```sql
select * from orders o,user u where o.uid=u.id
```

3. 辨析 resultMap
```xml
<resultMap id="orderMap" type="order">
    <!--手动指定字段与实体属性的映射关系
        column: 数据表的字段名称
        property：实体的属性名称
    -->
    <id column="oid" property="id"></id>
    <result column="ordertime" property="ordertime"></result>
    <result column="total" property="total"></result>
    <result column="uid" property="user.id"></result>
    <!--
        property: 当前实体(order)中的属性名称(private User user)
        javaType: 当前实体(order)中的属性的类型(User)
    -->
    <association property="user" javaType="user">
        <id column="uid" property="id"></id>
        <result column="username" property="username"></result>
        <result column="password" property="password"></result>
        <result column="birthday" property="birthday"></result>
    </association>
</resultMap>
```

### 一对多
如, 一个用户对应多张表, 可以使用 `collaction` 标签来解决

1. 编写实体类
```java
public class Order {    
	private int id;    
	private Date ordertime;    
	private double total;    
	//代表当前订单从属于哪一个客户    
	private User user;
	// 省略get set方法
}
```

```java
public class User {        
	private int id;    
	private String username;    
	private String password;    
	private Date birthday;
    //代表当前用户具备哪些订单    
    private List<Order> orderList;
    // 省略get set方法
}
```

2. 编写 sql 语句
```sql
SELECT *,o.id oid FROM USER u,orders o WHERE u.id=o.uid
```

3. 辨析 resultMap
```xml
<resultMap id="userMap" type="user">
    <id column="uid" property="id"></id>
    <result column="username" property="username"></result>
    <result column="password" property="password"></result>
    <result column="birthday" property="birthday"></result>
    <!--配置集合信息
        property:集合名称
        ofType：当前集合中的数据类型
    -->
    <collection property="orderList" ofType="order">
        <!--封装order的数据-->
        <id column="oid" property="id"></id>
        <result column="ordertime" property="ordertime"></result>
        <result column="total" property="total"></result>
    </collection>
</resultMap>

<select id="findAll" resultMap="userMap">
    SELECT *,o.id oid FROM USER u,orders o WHERE u.id=o.uid
</select>

```




## 整合 mybatis generator
1. 修改pom.xml文件引入 依赖和插件
```xml
<!-- 版本 -->
<mybatis-plus.version>3.5.2</mybatis-plus.version>  
<mybatis3.generator.plugin.version>0.4.0</mybatis3.generator.plugin.version>  
<mybatis-generator-core.version>1.3.7</mybatis-generator-core.version>  
<mysql.connector.version>8.0.31</mysql.connector.version>  
<mybatis.generator.version>1.3.5</mybatis.generator.version>

<!-- 依赖 -->
<dependency>  
  <groupId>com.baomidou</groupId>  
  <artifactId>mybatis-plus-boot-starter</artifactId>  
  <version>${mybatis-plus.version}</version>  
</dependency>  
  
<dependency>  
  <groupId>com.github.oceanc</groupId>  
  <artifactId>mybatis3-generator-plugin</artifactId>  
  <version>${mybatis3.generator.plugin.version}</version>  
</dependency>  
  
<dependency>  
  <groupId>org.mybatis.generator</groupId>  
  <artifactId>mybatis-generator-core</artifactId>  
  <version>${mybatis-generator-core.version}</version>  
</dependency>  
  
<dependency>  
  <groupId>com.mysql</groupId>  
  <artifactId>mysql-connector-j</artifactId>  
  <version>${mysql.connector.version}</version>  
</dependency>

<!-- 插件 -->
<plugin>  
  <groupId>org.mybatis.generator</groupId>  
  <artifactId>mybatis-generator-maven-plugin</artifactId>  
  <version>${mybatis.generator.version}</version>  
  <configuration>    
	<verbose>true</verbose>  
    <overwrite>true</overwrite>  
  </configuration>  
  <dependencies>
    <dependency>  
	   <groupId>com.mysql</groupId>  
       <artifactId>mysql-connector-j</artifactId>  
       <version>${mysql.connector.version}</version>  
    </dependency>  
    <dependency>      
      <groupId>com.fancier.prototype.common</groupId>  
      <artifactId>prototype-common</artifactId>  
      <version>${revision}</version>  
    </dependency>  
  </dependencies>  
</plugin>
```

2. 编写两个插件用来为生成的 entity 和 mapper 接口做修改
```java
// 为生成的 mapper 接口添加 extends BaseMapper 
public class BaseMapperGeneratorPlugin extends PluginAdapter {  
  
  public boolean validate(List<String> warnings) {  
    return true;  
  }  
  
  /**  
   * 生成dao  
   */  @Override  
  public boolean clientGenerated(Interface interfaze,  
                                 TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {  
    /**  
     * 主键默认采用java.lang.Integer  
     */    FullyQualifiedJavaType fqjt = new FullyQualifiedJavaType("BaseMapper<"  
        + introspectedTable.getBaseRecordType() + ">");  
    FullyQualifiedJavaType imp = new FullyQualifiedJavaType(  
        "com.baomidou.mybatisplus.core.mapper.BaseMapper");  
    /**  
     * 添加 extends BaseMapper  
     */    interfaze.addSuperInterface(fqjt);  
  
    /**  
     * 添加import com.baomidou.mybatisplus.core.mapper.BaseMapper;  
     */    interfaze.addImportedType(imp);  
    /**  
     * 方法不需要  
     */  
    interfaze.getMethods().clear();  
    interfaze.getAnnotations().clear();  
    return true;  
  }  
  
}
```

```java
// 为实体类添加 lombok 注解
public class LombokAnnotationPlugin extends PluginAdapter {  
  
    @Override  
    public boolean validate(List<String> warnings) {  
        return true;  
    }  
  
    @Override  
    public boolean modelBaseRecordClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {  
        topLevelClass.addAnnotation("@Data");  
        topLevelClass.addAnnotation("@AllArgsConstructor");  
        topLevelClass.addAnnotation("@NoArgsConstructor");  
        topLevelClass.addAnnotation("@Builder");  
        topLevelClass.addAnnotation("@TableName(\""+ introspectedTable.getFullyQualifiedTable() +"\")");  
        topLevelClass.addImportedType(new FullyQualifiedJavaType("lombok.Data"));  
        topLevelClass.addImportedType(new FullyQualifiedJavaType("lombok.AllArgsConstructor"));  
        topLevelClass.addImportedType(new FullyQualifiedJavaType("lombok.NoArgsConstructor"));  
        topLevelClass.addImportedType(new FullyQualifiedJavaType("lombok.Builder"));  
        topLevelClass.addImportedType(new FullyQualifiedJavaType("com.baomidou.mybatisplus.annotation.TableName"));  
  
  
  
        List<Method> methods =  topLevelClass.getMethods();  
        List<Method> remove = new ArrayList<>();  
        for (Method method : methods) {  
            if (method.getBodyLines().size() < 2) {  
                remove.add(method);  
                System.out.println("-----------------" + topLevelClass.getType().getShortName() + "'s method=" + method.getName() + " removed cause lombok annotation.");  
            }  
        }  
        methods.removeAll(remove);  
        return true;  
    }  
  
}
```

3. 编写配置文件,将两个插件配置下
```xml
<!DOCTYPE generatorConfiguration PUBLIC  
        "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"  
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">  
  
<generatorConfiguration>  
    <properties resource="mbg.properties"/>  
    <!-- MyBatis3Simple风格 -->  
    <context id="DB2Tables" targetRuntime="MyBatis3Simple">  
  
        <plugin type = "${plugin.package}.LombokAnnotationPlugin"/>  
        <plugin type = "${plugin.package}.BaseMapperGeneratorPlugin"/>  
  
        <commentGenerator>            <!-- 是否去除自动生成的注释 true：是 ： false:否。 自动生成注释太啰嗦，可以编码扩展CommentGenerator -->  
            <property name="suppressAllComments" value="true"/>  
        </commentGenerator>  
  
        <!-- 数据库连接 -->  
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"  
                        connectionURL="${jdbc.url}"  
                        userId="${jdbc.username}" password="${jdbc.password}">  
            <!-- 设置为true就只读取db_book下的表, 否则会优先读取到mysql的user表  -->  
            <property name="nullCatalogMeansCurrent" value="true"/>  
        </jdbcConnection>  
  
        <!-- 生成PO的包名和位置 -->  
        <javaModelGenerator targetPackage="${model.package}" targetProject="src/main/java">  
            <property name="useActualColumnNames" value="false"/>  
            <property name="enableLombok" value="true"/>  
        </javaModelGenerator>  
  
        <!-- 生成XML映射文件的包名和位置 -->  
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources"/>  
  
        <!-- 生成Mapper接口的包名和位置 -->  
        <javaClientGenerator type="XMLMAPPER" targetPackage="${client.package}" targetProject="src/main/java">  
            <property name="rootClass" value="com.baomidou.mybatisplus.core.mapper.BaseMapper"/>  
        </javaClientGenerator>  
  
        <table tableName="user" domainObjectName="User">  
            <!-- 自增主键列 -->  
            <generatedKey column="id" sqlStatement="JDBC" identity="true"/>  
            <!-- tinyint映射为Integer -->  
            <columnOverride column="is_delete" javaType="Integer" jdbcType="TINYINT"/>  
  
            <columnOverride column="create_time" javaType="LocalDateTime" jdbcType="TIMESTAMP"/>  
            <columnOverride column="update_time" javaType="LocalDateTime" jdbcType="TIMESTAMP"/>  
            <columnOverride column="edit_time" javaType="LocalDateTime" jdbcType="TIMESTAMP"/>  
        </table>  
    </context>  
</generatorConfiguration>
```

4. 在 resource 目录下编写properties配置文件, 用来配置数据库数据和实体类和 mapper 接口所在包数据
```properties
jdbc.url=jdbc:mysql://localhost:3307/xxx 
jdbc.username=root  
jdbc.password=123456  
model.package=com.fancier.prototype.common.model.entity  
client.package=com.fancier.prototype.common.mapper  
plugin.package=com.fancier.prototype.common.plugin
```

