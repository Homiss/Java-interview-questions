# Hibernate面试题

标签（空格分隔）： Java面试题

---
### 为什么要使用Hibernate开发你的项目呢？Hibernate的开发流程是怎么样的？
为什么要使用
①.对JDBC访问数据库的代码做了封装，大大简化了数据访问层繁琐的重复性代码。 
②.Hibernate 是一个基于JDBC的主流持久化框架，是一个优秀的ORM 实现。他很大程度的简化DAO层的编码工作 
③.hibernate 的性能非常好，因为它是个轻量级框架。映射的灵活性很出色。它支持各种关系数据库，从一对一到多对多的各种复杂关系。
开发流程
![image_1bmm5obo71dc04csqnn117q176o9.png-35.1kB][1]

### 什么是延迟加载？
    延迟加载机制是为了避免一些无谓的性能开销而提出来的，所谓延迟加载就是当在真正需要数据的时候，才真正执行数据加载操作。在Hibernate中提供了对实体对象的延迟加载以及对集合的延迟加载，另外在Hibernate3中还提供了对属性的延迟加载。

### 说一下hibernate的缓存机制
A：hibernate一级缓存 
（1）hibernate支持两个级别的缓存，默认只支持一级缓存； 
（2）每个Session内部自带一个一级缓存； 
（3）某个Session被关闭时，其对应的一级缓存自动清除； 
B：hibernate二级缓存 
(1) 二级缓存独立于session，默认不开启；

### Hibernate的查询方式有哪些？
本地SQL查询、Criteria、Hql

### 如何优化Hibernate？
1.使用双向一对多关联，不使用单向一对多
2.灵活使用单向一对多关联
3.不用一对一，用多对一取代
4.配置对象缓存，不使用集合缓存
5.一对多集合使用Bag,多对多集合使用Set
6. 继承类使用显式多态
7. 表字段要少，表关联不要怕多，有二级缓存撑腰 


### Hibernate中GET和LOAD的区别？
把get和load放到一起进行对比是Hibernate面试时最常问到的问题，这是因为只有正确理解get()和load()这二者后才有可能高效地使用Hibernate。get和load的最大区别是，如果在缓存中没有找到相应的对象，get将会直接访问数据库并返回一个完全初始化好的对象，而这个过程有可能会涉及到多个数据库调用；而load方法在缓存中没有发现对象的情况下，只会返回一个代理对象，只有在对象getId()之外的其它方法被调用时才会真正去访问数据库，这样就能在某些情况下大幅度提高性能。

### 说说在 hibernate中使用Integer做映射和使用int做映射之间有什么差别？
Integer code和int code的区别: 
> Integer是对象.      code=null;   对象可以为空.    
int 是普通类型,     不可能=null.        

根据你的数据库code是可以空的,故应该映射成Integer.       
你没理由hbm.xml里写 Integer,类里却写int

### SQL和HQL有什么区别？
sql 面向数据库表查询 
hql 面向对象查询 
  
hql：from 后面跟的 类名＋类对象 where 后 用 对象的属性做条件 
sql：from 后面跟的是表名  where 后 用表中字段做条件 
查询 
在Hibernate中使用查询时，一般使用Hql查询语句。 
HQL（Hibernate Query Language），即Hibernate的查询语言跟SQL非常相像。不过HQL与SQL的最根本的区别，就是它是面向对象的。 

使用HQL时需要注意以下几点： 
1.大小写敏感 
> 因为HQL是面向对象的，而对象类的名称和属性都是大小写敏感的，所以HQL是大小写敏感的。
HQL语句：from Cat as cat where cat.id > 1;与from Cat as cat where cat.ID > 1;是不一样的，这点与SQL不同。

2.from子句 
> from Cat，该句返回Cat对象实例，开发人员也可以给其加上别名，eg. from Cat as cat，对于多表查询的情况，可参考如下：
from Cat as cat, Dog as dog
其它方面都与SQL类似，在此不再赘述。

### Hibernate的分页查询
例如：从数据库中的第20000条数据开始查后面100条记录 
Query q = session.createQuery("from Cat as c");;    
q.setMaxResults(100);;    
List l = q.list();; 
q.setFirstResult(20000);;  

### Hibernate中Java对象的状态以及对应的特征有哪些？
持久化对象的三种状态 
    持久化对象PO和OID 
PO=POJO + hbm映射配置 
    编写规则 
①必须提供无参数public构造器
②所有属性private，提供public的getter和setter方法
③必须提供标识属性，与数据表中主键对应，例如Customer类 id属性
④PO类属性应尽量使用基本数据类型的包装类型(区分空值)  例如int---Integer  long---Long
⑤不要用final修饰（将无法生成代理对象进行优化）
OID 指与数据表中主键对应 PO类中属性，例如 Customer类 id属性

Hibernate框架使用OID来区分不同PO对象 
* 例如内存中有两个PO对象，只要具有相同 OID， Hibernate认为同一个对象 
* Hibernate 不允许缓存同样OID的两个不同对象
     
①瞬时态(临时态、自由态)：不存在持久化标识OID，尚未与Hibernate Session关联对象，被认为处于瞬时态，失去引用将被JVM回收 
②持久态：存在持久化标识OID，与当前session有关联，并且相关联的session没有关闭 ,并且事务未提交 
③脱管态(离线态、游离态)：存在持久化标识OID，但没有与当前session关联，脱管状态改变hibernate不能检测到

区分三种状态：判断对象是否有OID，判断对象是否与session关联(被一级缓存引用) 
```
// 获得Session
Session session =HibernateUtils.openSession();
// 开启事务
Transaction transaction = session.beginTransaction();
 
Book book =newBook();// 瞬时态(没有OID，未与Session关联)
book.setName("hibernate精通");
book.setPrice(56d);
 
session.save(book);// 持久态(具有OID，与Session关联)
 
// 提交事务，关闭Session
transaction.commit();
session.close();
 
System.out.println(book.getId());// 脱管态(具有 OID，与Session断开关联)
```
持久化对象状态转换
![image_1bmm60o0nh1qsrk1b8bv3s1mqg1m.png-95.9kB][2]
①瞬时态对象：通过new获得 
瞬时----->持久    save、saveOrUpdate(都是session)
瞬时----->脱管    book.setId(1) 为瞬时对象设置OID 

②持久态对象：通过get/load 、Query查询获得
持久----->瞬时    delete  （被删除持久化对象 不建议再次使用 ）
持久----->脱管    evict(清除一级缓存中某一个对象)、close(关闭Session，清除一级缓存)、clear(清除一级缓存所有对象 )
 
③脱管态对象 无法直接获得
脱管----->瞬时    book.setId(null); 删除对象OID
脱管----->持久    update、saveOrUpdate、 lock(过时)

### Hibernate中怎样处理事务?

### Hibernate中save、persist和saveOrUpdate这三个方法的不同之处？
除了get和load，这又是另外一个经常出现的Hibernate面试问题。 所有这三个方法，也就是save()、saveOrUpdate()和persist()都是用于将对象保存到数据库中的方法，但其中有些细微的差别。例如，save()只能INSERT记录，但是saveOrUpdate()可以进行 记录的INSERT和UPDATE。还有，save()的返回值是一个Serializable对象，而persist()方法返回值为void。

### Hibernate中的命名SQL查询指的是什么? 
Hibernate的这个面试问题同Hibernate提供的查询功能相关。命名查询指的是用<sql-query>标签在影射文档中定义的SQL查询，可以通过使用Session.getNamedQuery()方法对它进行调用。命名查询使你可以使用你所指定的一个名字拿到某个特定的查询。 Hibernate中的命名查询可以使用注解来定义，也可以使用我前面提到的xml影射问句来定义。在Hibernate中，@NameQuery用来定义单个的命名查询，@NameQueries用来定义多个命名查询。 

### Hibernate中的SessionFactory有什么作用? SessionFactory是线程安全的吗？ 
这也是Hibernate框架的常见面试问题。顾名思义，SessionFactory就是一个用于创建Hibernate的Session对象的工厂。SessionFactory通常是在应用启动时创建好的，应用程序中的代码用它来获得Session对象。作为一个单个的数据存储，它也是 线程安全的，所以多个线程可同时使用同一个SessionFactory。Java JEE应用一般只有一个SessionFactory，服务于客户请求的各线程都通过这个工厂来获得Hibernate的Session实例，这也是为什么SessionFactory接口的实现必须是线程安全的原因。还有，SessionFactory的内部状态包含着同对象关系影射有关的所有元数据，它是 不可变的，一旦创建好后就不能对其进行修改了。

### Hibernate中的Session指的是什么? 可否将单个的Session在多个线程间进行共享？ 
Session代表着Hibernate所做的一小部分工作，它负责维护者同数据库的链接而且 不是线程安全的，也就是说，Hibernage中的Session不能在多个线程间进行共享。虽然Session会以主动滞后的方式获得数据库连接，但是Session最好还是在用完之后立即将其关闭。 

### Hibernate中sorted collection和ordered collection有什么不同?
sorted collection是通过使用 Java的Comparator在内存中进行排序的，ordered collection中的排序用的是数据库的order by子句。对于比较大的数据集，为了避免在内存中对它们进行排序而出现 Java中的OutOfMemoryError，最好使用ordered collection。

### Hibernate中transient、persistent、detached对象三者之间有什么区别？ 
在Hibernate中，对象具有三种状态：transient、persistent和detached。同Hibernate的session有关联的对象是persistent对象。对这种对象进行的所有修改都会按照事先设定的刷新策略，反映到数据库之中，也即，可以在对象的任何一个属性发生改变时自动刷新，也可以通过调用Session.flush()方法显式地进行刷新。如果一个对象原来同Session有关联关系，但当下却没有关联关系了，这样的对象就是detached的对象。你可以通过调用任意一个session的update()或者saveOrUpdate()方法，重新将该detached对象同相应的seesion建立关联关系。Transient对象指的是新建的持久化类的实例，它还从未同Hibernate的任何Session有过关联关系。同样的，你可以调用persist()或者save()方法，将transient对象变成persistent对象。可要记住，这里所说的transient指的可不是 Java中的transient关键字，二者风马牛不相及。

### Hibernate中Session的lock()方法有什么作用?
这是一个比较棘手的Hibernate面试问题，因为Session的lock()方法重建了关联关系却并没有同数据库进行同步和更新。因此，你在使用lock()方法时一定要多加小心。顺便说一下，在进行关联关系重建时，你可以随时使用Session的update()方法同数据库进行同步。有时这个问题也可以这么来问：Session的lock()方法和update()方法之间有什么区别？。这个小节中的关键点也可以拿来回答这个问题。

### Hibernate中二级缓存指的是什么？
这是同Hibernate的缓存机制相关的第一个面试问题，不出意外后面还会有更多这方面的问题。二级缓存是在SessionFactory这个级别维护的缓存，它能够通过节省几番数据库调用往返来提高性能。还有一点值得注意，二级缓存是针对整个应用而不是某个特定的session的。

### Hibernate中的查询缓存指的是什么？
这个问题有时是作为上个Hibernate面试问题的后继问题提出的。查询缓存实际上保存的是sql查询的结果，这样再进行相同的sql查询就可以之间从缓存中拿到结果了。为了改善性能，查询缓存可以同二级缓存一起来使用。Hibernate支持用多种不同的开源缓存方案，比如EhCache，来实现查询缓存。

### 为什么在Hibernate的实体类中要提供一个无参数的构造器这一点非常重要？

每个Hibernate实体类必须包含一个 无参数的构造器, 这是因为Hibernate框架要使用Reflection API，通过调用Class.newInstance()来创建这些实体类的实例。如果在实体类中找不到无参数的构造器，这个方法就会抛出一个InstantiationException异常。

### 可不可以将Hibernate的实体类定义为final类? 
是的，你可以将Hibernate的实体类定义为final类，但这种做法并不好。因为Hibernate会使用代理模式在延迟关联的情况下提高性能，如果你把实体类定义成final类之后，因为 Java不允许对final类进行扩展，所以Hibernate就无法再使用代理了，如此一来就限制了使用可以提升性能的手段。不过，如果你的持久化类实现了一个接口而且在该接口中声明了所有定义于实体类中的所有public的方法轮到话，你就能够避免出现前面所说的不利后果。 


  [1]: http://static.zybuluo.com/homiss/4rmst5h4dtcopryo5qocajx8/image_1bmm5obo71dc04csqnn117q176o9.png
  [2]: http://static.zybuluo.com/homiss/uz4wnhan875tsw19vxtkpedc/image_1bmm60o0nh1qsrk1b8bv3s1mqg1m.png