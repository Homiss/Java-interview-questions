# MyBatis面试题

标签（空格分隔）： Java面试题

---
### JDBC编程有哪些不足之处，MyBatis是如何解决这些问题的？
① 数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。
解决：在SqlMapConfig.xml中配置数据链接池，使用连接池管理数据库链接。

② Sql语句写在代码中造成代码不易维护，实际应用sql变化的可能较大，sql变动需要改变java代码。
解决：将Sql语句配置在XXXXmapper.xml文件中与java代码分离。

③ 向sql语句传参数麻烦，因为sql语句的where条件不一定，可能多也可能少，占位符需要和参数一一对应。
解决： Mybatis自动将java对象映射至sql语句。

④ 对结果集解析麻烦，sql变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成pojo对象解析比较方便。
解决：Mybatis自动将sql执行结果映射至java对象。
### MyBatis编程步骤是什么样的？
① 创建SqlSessionFactory 
② 通过SqlSessionFactory创建SqlSession 
③ 通过sqlsession执行数据库操作 
④ 调用session.commit()提交事务 
⑤ 调用session.close()关闭会话

### MyBatis与Hibernate有哪些不同？
Mybatis和hibernate不同，它不完全是一个ORM框架，因为MyBatis需要程序员自己编写Sql语句，不过mybatis可以通过XML或注解方式灵活配置要运行的sql语句，并将java对象和sql语句映射生成最终执行的sql，最后将sql执行的结果再映射生成java对象。 
  
Mybatis学习门槛低，简单易学，程序员直接编写原生态sql，可严格控制sql执行性能，灵活度高，非常适合对关系数据模型要求不高的软件开发，例如互联网软件、企业运营类软件等，因为这类软件需求变化频繁，一但需求变化要求成果输出迅速。但是灵活的前提是mybatis无法做到数据库无关性，如果需要实现支持多种数据库的软件则需要自定义多套sql映射文件，工作量大。 
  
Hibernate对象/关系映射能力强，数据库无关性好，对于关系模型要求高的软件（例如需求固定的定制化软件）如果用hibernate开发可以节省很多代码，提高效率。但是Hibernate的缺点是学习门槛高，要精通门槛更高，而且怎么设计O/R映射，在性能和对象模型之间如何权衡，以及怎样用好Hibernate需要具有很强的经验和能力才行。 
总之，按照用户的需求在有限的资源环境下只要能做出维护性、扩展性良好的软件架构都是好架构，所以框架只有适合才是最好。

### 使用MyBatis的mapper接口调用时有哪些要求？
①  Mapper接口方法名和mapper.xml中定义的每个sql的id相同 
②  Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同 
③  Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同 
④  Mapper.xml文件中的namespace即是mapper接口的类路径。
5.SqlMapConfig.xml中配置有哪些内容？
SqlMapConfig.xml中配置的内容和顺序如下： 
properties（属性）
settings（配置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境集合属性对象）
environment（环境子属性对象）
transactionManager（事务管理）
dataSource（数据源）
mappers（映射器）

### 简单的说一下MyBatis的一级缓存和二级缓存？
Mybatis首先去缓存中查询结果集，如果没有则查询数据库，如果有则从缓存取出返回结果集就不走数据库。Mybatis内部存储缓存使用一个HashMap，key为hashCode+sqlId+Sql语句。value为从查询出来映射生成的java对象

Mybatis的二级缓存即查询缓存，它的作用域是一个mapper的namespace，即在同一个namespace中查询sql可以从缓存中获取数据。二级缓存是可以跨SqlSession的。

### Mapper编写有哪几种方式？
①接口实现类继承SqlSessionDaoSupport
使用此种方法需要编写mapper接口，mapper接口实现类、mapper.xml文件

1、在sqlMapConfig.xml中配置mapper.xml的位置
```
<mappers>
    <mapper resource="mapper.xml文件的地址" />
    <mapper resource="mapper.xml文件的地址" />
</mappers>
```

2、定义mapper接口

3、实现类集成SqlSessionDaoSupport

mapper方法中可以this.getSqlSession()进行数据增删改查。

4、spring 配置
```
<bean id=" " class="mapper接口的实现">
    <property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
</bean>
```
②使用org.mybatis.spring.mapper.MapperFactoryBean
 

1、在sqlMapConfig.xml中配置mapper.xml的位置

如果mapper.xml和mappre接口的名称相同且在同一个目录，这里可以不用配置
```
<mappers>
    <mapper resource="mapper.xml文件的地址" />
    <mapper resource="mapper.xml文件的地址" />
</mappers>
```

2、定义mapper接口

注意

1、mapper.xml中的namespace为mapper接口的地址

2、mapper接口中的方法名和mapper.xml中的定义的statement的id保持一致

3、 Spring中定义
```
<bean id="" class="org.mybatis.spring.mapper.MapperFactoryBean">
    <property name="mapperInterface"   value="mapper接口地址" /> 
    <property name="sqlSessionFactory" ref="sqlSessionFactory" /> 
</bean>
```
③使用mapper扫描器
 

1、mapper.xml文件编写

> 注意：
mapper.xml中的namespace为mapper接口的地址
mapper接口中的方法名和mapper.xml中的定义的statement的id保持一致
如果将mapper.xml和mapper接口的名称保持一致则不用在sqlMapConfig.xml中进行配置 

2、定义mapper接口

> 注意mapper.xml的文件名和mapper的接口名称保持一致，且放在同一个目录

3、配置mapper扫描器
```
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="mapper接口包地址"></property>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/> 
</bean>
```
4、使用扫描器后从spring容器中获取mapper的实现对象扫描器将接口通过代理方法生成实现对象，要spring容器中自动注册，名称为mapper 接口的名称。

### 结束

参考：
[[Java面试七]Mybatis总结以及在面试中的一些问题.][1]


  [1]: http://www.cnblogs.com/wang-meng/p/5701990.html