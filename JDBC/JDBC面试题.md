# JDBC面试题

标签（空格分隔）： Java面试题 JDBC

---

### 什么是JDBC，在什么时候会用到它？
JDBC的全称是Java DataBase Connection，也就是Java数据库连接，我们可以用它来操作关系型数据库。JDBC接口及相关类在java.sql包和javax.sql包里。我们可以用它来连接数据库，执行SQL查询，存储过程，并处理返回的结果。

JDBC接口让Java程序和JDBC驱动实现了松耦合，使得切换不同的数据库变得更加简单。

### 有哪些不同类型的JDBC驱动？
有四类JDBC驱动。和数据库进行交互的Java程序分成两个部分，一部分是JDBC的API，实际工作的驱动则是另一部分。
![image_1blq868ecc3bguks6o17ro1eo919.png-275.8kB][1]

A JDBC-ODBC Bridge plus ODBC Driver（类型1）：它使用ODBC驱动连接数据库。需要安装ODBC以便连接数据库，正因为这样，这种方式现在已经基本淘汰了。

B Native API partly Java technology-enabled driver（类型2）：这种驱动把JDBC调用适配成数据库的本地接口的调用。

C Pure Java Driver for Database Middleware（类型3）：这个驱动把JDBC调用转发给中间件服务器，由它去和不同的数据库进行连接。用这种类型的驱动需要部署中间件服务器。这种方式增加了额外的网络调用，导致性能变差，因此很少使用。

D Direct-to-Database Pure Java Driver（类型4）：这个驱动把JDBC转化成数据库使用的网络协议。这种方案最简单，也适合通过网络连接数据库。不过使用这种方式的话，需要根据不同数据库选用特定的驱动程序，比如OJDBC是Oracle开发的Oracle数据库的驱动，而MySQL Connector/J是MySQL数据库的驱动。

### JDBC是如何实现Java程序和JDBC驱动的松耦合的？
JDBC API使用Java的反射机制来实现Java程序和JDBC驱动的松耦合。随便看一个简单的JDBC示例，你会发现所有操作都是通过JDBC接口完成的，而驱动只有在通过Class.forName反射机制来加载的时候才会出现。

### 什么是JDBC连接，在Java中如何创建一个JDBC连接？
JDBC连接是和数据库服务器建立的一个会话。你可以想像成是一个和数据库的Socket连接。

创建JDBC连接很简单，只需要两步：

A. 注册并加载驱动：使用Class.forName()，驱动类就会注册到DriverManager里面并加载到内存里。 
B. 用DriverManager获取连接对象：调用DriverManager.getConnnection()方法并传入数据库连接的URL，用户名及密码，就能获取到连接对象。

```java
Connection con = null;
try{
    // load the Driver Class
    Class.forName("com.mysql.jdbc.Driver");
    // create the connection now
    con = DriverManager.getConnection("jdbc:mysql://localhost:3306/UserDB",
                    "pankaj",
                    "pankaj123");
    }catch (SQLException e) {
            System.out.println("Check database is UP and configs are correct");
            e.printStackTrace();
    }catch (ClassNotFoundException e) {
            System.out.println("Please include JDBC MySQL jar in classpath");
            e.printStackTrace();
}
```

### 阐述JDBC操作数据库的步骤?

加载驱动。
```
Class.forName("oracle.jdbc.driver.OracleDriver");
```
创建连接。
```
Connection con = DriverManager.getConnection("jdbc:oracle:thin:@localhost:1521:orcl", "scott", "tiger");
```
创建语句。
```
PreparedStatement ps = con.prepareStatement("select * from emp where sal between ? and ?");
    ps.setInt(1, 1000);
    ps.setInt(2, 3000);
```
执行语句。
```
ResultSet rs = ps.executeQuery();
```
处理结果。
```
while(rs.next()) {
        System.out.println(rs.getInt("empno") + " - " + rs.getString("ename"));
}
```
关闭资源。
```
finally {
    if(con != null) {
        try {
            con.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### 使用JDBC操作数据库时，如何提升读取数据的性能？如何提升更新数据的性能？ 
要提升读取数据的性能，可以指定通过结果集（ResultSet）对象的setFetchSize()方法指定每次抓取的记录数（典型的空间换时间策略）；
要提升更新数据的性能可以使用PreparedStatement语句构建批处理，将若干SQL语句置于一个批处理中执行。

### JDBC的DriverManager是用来做什么的？
JDBC的DriverManager是一个工厂类，我们通过它来创建数据库连接。当JDBC的Driver类被加载进来时，它会自己注册到DriverManager类里面。

然后我们会把数据库配置信息传成DriverManager.getConnection()方法，DriverManager会使用注册到它里面的驱动来获取数据库连接，并返回给调用的程序。

### 在进行数据库编程时，连接池有什么作用？ 
由于创建连接和释放连接都有很大的开销（尤其是数据库服务器不在本地时，每次建立连接都需要进行TCP的三次握手，释放连接需要进行TCP四次握手，造成的开销是不可忽视的），为了提升系统访问数据库的性能，可以事先创建若干连接置于连接池中，需要时直接从连接池获取，使用结束时归还连接池而不必关闭连接，从而避免频繁创建和释放连接所造成的开销，这是典型的用空间换取时间的策略（浪费了空间存储连接，但节省了创建和释放连接的时间）。

池化技术在Java开发中是很常见的，在使用线程时创建线程池的道理与此相同。

### 什么是DAO模式？ 
DAO（Data Access Object）顾名思义是一个为数据库或其他持久化机制提供了抽象接口的对象，在不暴露底层持久化方案实现细节的前提下提供了各种数据访问操作。

在实际的开发中，应该将所有对数据源的访问操作进行抽象化后封装在一个公共API中。用程序设计语言来说，就是建立一个接口，接口中定义了此应用程序中将会用到的所有事务方法。在这个应用程序中，当需要和数据源进行交互的时候则使用这个接口，并且编写一个单独的类来实现这个接口，在逻辑上该类对应一个特定的数据存储。

DAO模式实际上包含了两个模式，一是Data Accessor（数据访问器），二是Data Object（数据对象），前者要解决如何访问数据的问题，而后者要解决的是如何用对象封装数据。

### JDBC的Statement是什么？
Statement是JDBC中用来执行数据库SQL查询语句的接口。通过调用连接对象的getStatement()方法我们可以生成一个Statement对象。我们可以通过调用它的execute()，executeQuery()，executeUpdate()方法来执行静态SQL查询。

默认情况下，一个Statement同时只能打开一个ResultSet。如果想操作多个ResultSet对象的话，需要创建多个Statement。Statement接口的所有execute方法开始执行时都默认会关闭当前打开的ResultSet。

### execute，executeQuery，executeUpdate的区别是什么？
execute(String query)方法用来执行任意的SQL查询，如果查询的结果是一个ResultSet，这个方法就返回true。如果结果不是ResultSet，比如insert或者update查询，它就会返回false。我们可以通过它的getResultSet方法来获取ResultSet，或者通过getUpdateCount()方法来获取更新的记录条数。

executeQuery(String query)接口用来执行select查询，并且返回ResultSet。即使查询不到记录返回的ResultSet也不会为null。我们通常使用executeQuery来执行查询语句，这样的话如果传进来的是insert或者update语句的话，它会抛出错误信息为 “executeQuery method can not be used for update”的java.util.SQLException。

executeUpdate(String query)方法用来执行insert或者update/delete（DML）语句，或者 什么也不返回DDL语句。返回值是int类型，如果是DML语句的话，它就是更新的条数，如果是DDL的话，就返回0。

只有当你不确定是什么语句的时候才应该使用execute()方法，否则应该使用executeQuery或者executeUpdate方法。

### JDBC的PreparedStatement是什么？
PreparedStatement对象代表的是一个预编译的SQL语句。用它提供的setter方法可以传入查询的变量。

由于PreparedStatement是预编译的，通过它可以将对应的SQL语句高效的执行多次。由于PreparedStatement自动对特殊字符转义，避免了SQL注入攻击，因此应当尽量的使用它。

### PreparedStatement中如何注入NULL值？
可以使用它的setNull方法来把null值绑定到指定的变量上。setNull方法需要传入参数的索引以及SQL字段的类型，像这样：
```java
ps.setNull(10, java.sql.Types.INTEGER);
```

### Statement中的getGeneratedKeys方法有什么用？
有的时候表会生成主键，这时候就可以用Statement的getGeneratedKeys()方法来获取这个自动生成的主键的值了。

### 相对于Statement，PreparedStatement的优点是什么？
它和Statement相比优点在于：

1. 有助于防止SQL注入，因为它会自动对特殊字符转义。
2. 可以用来进行动态查询。
3. 执行更快。尤其当你重用它或者使用它的拼量查询接口执行多条语句时。

使用PreparedStatement的setter方法更容易写出面向对象的代码，而Statement的话，我们得拼接字符串来生成查询语句。如果参数太多了，字符串拼接看起来会非常丑陋并且容易出错。

### PreparedStatement的缺点是什么，怎么解决这个问题？
PreparedStatement的一个缺点是，我们不能直接用它来执行in条件语句；需要执行IN条件语句的话，下面有一些解决方案：

- 分别进行单条查询——这样做性能很差，不推荐。
- 使用存储过程——这取决于数据库的实现，不是所有数据库都支持。
- 动态生成PreparedStatement——这是个好办法，但是不能享受PreparedStatement的缓存带来的好处了。
- 在PreparedStatement查询中使用NULL值——如果你知道输入变量的最大个数的话，这是个不错的办法，扩展一下还可以支持无限参数。

### JDBC的ResultSet是什么？
在查询数据库后会返回一个ResultSet，它就像是查询结果集的一张数据表。

ResultSet对象维护了一个游标，指向当前的数据行。开始的时候这个游标指向的是第一行。如果调用了ResultSet的next()方法游标会下移一行，如果没有更多的数据了，next()方法会返回false。可以在for循环中用它来遍历数据集。

默认的ResultSet是不能更新的，游标也只能往下移。也就是说你只能从第一行到最后一行遍历一遍。不过也可以创建可以回滚或者可更新的ResultSet，像下面这样。
```java
Statement stmt = con.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE,
                                   ResultSet.CONCUR_UPDATABLE);
```
当生成ResultSet的Statement对象要关闭或者重新执行或是获取下一个ResultSet的时候，ResultSet对象也会自动关闭。

可以通过ResultSet的getter方法，传入列名或者从1开始的序号来获取列数据。

### 有哪些不同的ResultSet？
根据创建Statement时输入参数的不同，会对应不同类型的ResultSet。如果你看下Connection的方法，你会发现createStatement和prepareStatement方法重载了，以支持不同的ResultSet和并发类型。

一共有三种ResultSet对象。

- ResultSet.TYPE_FORWARD_ONLY：这是默认的类型，它的游标只能往下移。
- ResultSet.TYPE_SCROLL_INSENSITIVE：游标可以上下移动，一旦它创建后，数据库里的数据再发生修改，对它来说是透明的。
- ResultSet.TYPE_SCROLL_SENSITIVE：游标可以上下移动，如果生成后数据库还发生了修改操作，它是能够感知到的。
ResultSet有两种并发类型。

- ResultSet.CONCUR_READ_ONLY:ResultSet是只读的，这是默认类型。
- ResultSet.CONCUR_UPDATABLE:我们可以使用ResultSet的更新方法来更新里面的数据。

### Statement中的setFetchSize和setMaxRows方法有什么用处？
setMaxRows可以用来限制返回的数据集的行数。当然通过SQL语句也可以实现这个功能。比如在MySQL中我们可以用LIMIT条件来设置返回结果的最大行数。

setFetchSize理解起来就有点费劲了，因为你得知道Statement和ResultSet是怎么工作的。当数据库在执行一条查询语句时，查询到的数据是在数据库的缓存中维护的。ResultSet其实引用的是数据库中缓存的结果。

假设我们有一条查询返回了100行数据，我们把fetchSize设置成了10，那么数据库驱动每次只会取10条数据，也就是说得取10次。当每条数据需要处理的时间比较长的时候并且返回数据又非常多的时候，这个可选的参数就变得非常有用了。

我们可以通过Statement来设置fetchSize参数，不过它会被ResultSet对象设置进来的值所覆盖掉。

### 如何使用JDBC接口来调用存储过程？
存储过程就是数据库编译好的一组SQL语句，可以通过JDBC接口来进行调用。我们可以通过JDBC的CallableStatement接口来在数据库中执行存储过程。初始化CallableStatement的语法是这样的：
```java
CallableStatement stmt = con.prepareCall("{call insertEmployee(?,?,?,?,?,?)}");
stmt.setInt(1, id);
stmt.setString(2, name);
stmt.setString(3, role);
stmt.setString(4, city);
stmt.setString(5, country);
//register the OUT parameter before calling the stored procedure
stmt.registerOutParameter(6, java.sql.Types.VARCHAR);
stmt.executeUpdate();
```
我们得在执行CallableStatement之前注册OUT参数。

### JDBC的批处理是什么，有什么好处？
有时候类似的查询我们需要执行很多遍，比如从CSV文件中加载数据到关系型数据库的表里。我们也知道，执行查询可以用Statement或者PreparedStatement。除此之外，JDBC还提供了批处理的特性，有了它，我们可以在一次数据库调用中执行多条查询语句。

JDBC通过Statement和PreparedStatement中的addBatch和executeBatch方法来支持批处理。

批处理比一条条语句执行的速度要快得多，因为它需要很少的数据库调用。

### JDBC的事务管理是什么，为什么需要它？
默认情况下，我们创建的数据库连接，是工作在自动提交的模式下的。这意味着只要我们执行完一条查询语句，就会自动进行提交。因此我们的每条查询，实际上都是一个事务，如果我们执行的是DML或者DDL，每条语句完成的时候，数据库就已经完成修改了。

有的时候我们希望由一组SQL查询组成一个事务，如果它们都执行OK我们再进行提交，如果中途出现异常了，我们可以进行回滚。

JDBC接口提供了一个setAutoCommit(boolean flag)方法，我们可以用它来关闭连接自动提交的特性。我们应该在需要手动提交时才关闭这个特性，不然的话事务不会自动提交，每次都得手动提交。数据库通过表锁来管理事务，这个操作非常消耗资源。因此我们应当完成操作后尽快的提交事务。

### 如何回滚事务？
通过Connection对象的rollback方法可以回滚事务。它会回滚这次事务中的所有修改操作，并释放当前连接所持有的数据库锁。

### JDBC的保存点(Savepoint)是什么，如何使用？
有时候事务包含了一组语句，而我们希望回滚到这个事务的某个特定的点。JDBC的保存点可以用来生成事务的一个检查点，使得事务可以回滚到这个检查点。

一旦事务提交或者回滚了，它生成的任何保存点都会自动释放并失效。回滚事务到某个特定的保存点后，这个保存点后所有其它的保存点会自动释放并且失效。

### JDBC的DataSource是什么，有什么好处？
DataSource即数据源，它是定义在javax.sql中的一个接口，跟DriverManager相比，它的功能要更强大。我们可以用它来创建数据库连接，当然驱动的实现类会实际去完成这个工作。除了能创建连接外，它还提供了如下的特性：

- 缓存PreparedStatement以便更快的执行
- 可以设置连接超时时间
- 提供日志记录的功能
- ResultSet大小的最大阈值设置
- 通过JNDI的支持，可以为servlet容器提供连接池的功能

### 什么是数据库的隔离级别？
当我们为了数据的一致性使用事务时，数据库系统用锁来防止别人访问事务中用到的数据。数据库通过锁来防止脏读，不可重复读(Non-Repeatable Reads)及幻读（Phantom-Read）的问题。

数据库使用JDBC设置的隔离级别来决定它使用何种锁机制，我们可以通过Connection的getTransactionIsolation和setTransactionIsolation方法来获取和设置数据库的隔离级别。

|隔离级别 |	事务 |	脏读	| 不可重复读 |	幻读 |
| --------   | -----  | -----  | -----  | -----  |
|TRANSACTION_NONE|	不支持	|不可用|	不可用|	不可用|
|TRANSACTION_READ_COMMITTED|	支持|	阻止|	允许|	允许|
|TRANSACTION_READ_UNCOMMITTED	|支持	|允许	|允许|	允许|
|TRANSACTION_REPEATABLE_READ	|支持	|阻止	|阻止	|允许|
|TRANSACTION_SERIALIZABLE	|支持	|阻止	|阻止	|阻止|

### JDBC的RowSet是什么，有哪些不同的RowSet？
RowSet用于存储查询的数据结果，和ResultSet相比，它更具灵活性。RowSet继承自ResultSet，因此ResultSet能干的，它们也能，而ResultSet做不到的，它们还是可以。RowSet接口定义在javax.sql包里。

RowSet提供的额外的特性有：

- 提供了Java Bean的功能，可以通过settter和getter方法来设置和获取属性。RowSet使用了JavaBean的事件驱动模型，它可以给注册的组件发送事件通知，比如游标的移动，行的增删改，以及RowSet内容的修改等。
- RowSet对象默认是可滚动，可更新的，因此如果数据库系统不支持ResultSet实现类似的功能，可以使用RowSet来实现。
RowSet分为两大类：

A. 连接型RowSet——这类对象与数据库进行连接，和ResultSet很类似。JDBC接口只提供了一种连接型RowSet，javax.sql.rowset.JdbcRowSet，它的标准实现是com.sun.rowset.JdbcRowSetImpl。 B. 离线型RowSet——这类对象不需要和数据库进行连接，因此它们更轻量级，更容易序列化。它们适用于在网络间传递数据。有四种不同的离线型RowSet的实现。

- CachedRowSet——可以通过他们获取连接，执行查询并读取ResultSet的数据到RowSet里。我们可以在离线时对数据进行维护和更新，然后重新连接到数据库里，并回写改动的数据。
- WebRowSet继承自CachedRowSet——他可以读写XML文档。
- JoinRowSet继承自WebRowSet——它不用连接数据库就可以执行SQL的join操作。
- FilteredRowSet继承自WebRowSet——我们可以用它来设置过滤规则，这样只有选中的数据才可见。

### RowSet和ResultSet的区别是什么？
RowSet继承自ResultSet，因此它有ResultSet的全部功能，同时它自己添加了些额外的特性。RowSet一个最大的好处是它可以是离线的，这样使得它更轻量级，同时便于在网络间进行传输。

具体使用哪个取决于你的需求，不过如果你操作ResultSet对象的时间较长的话，最好选择一个离线的RowSet，这样可以释放数据库连接。

### 常见的JDBC异常有哪些？
有以下这些：

- java.sql.SQLException——这是JDBC异常的基类。
- java.sql.BatchUpdateException——当批处理操作执行失败的时候可能会抛出这个异常。这取决于具体的JDBC驱动的实现，它也可能直接抛出基类异常java.sql.SQLException。
- java.sql.SQLWarning——SQL操作出现的警告信息。
- java.sql.DataTruncation——字段值由于某些非正常原因被截断了（不是因为超过对应字段类型的长度限制）。

### JDBC里的CLOB和BLOB数据类型分别代表什么？
CLOB意思是Character Large OBjects，字符大对象，它是由单字节字符组成的字符串数据，有自己专门的代码页。这种数据类型适用于存储超长的文本信息，那些可能会超出标准的VARCHAR数据类型长度限制（上限是32KB）的文本。

BLOB是Binary Larget OBject，它是二进制大对象，由二进制数据组成，没有专门的代码页。它能用于存储超过VARBINARY限制（32KB）的二进制数据。这种数据类型适合存储图片，声音，图形，或者其它业务程序特定的数据。

### JDBC的脏读是什么？哪种数据库隔离级别能防止脏读？
当我们使用事务时，有可能会出现这样的情况，有一行数据刚更新，与此同时另一个查询读到了这个刚更新的值。这样就导致了脏读，因为更新的数据还没有进行持久化，更新这行数据的业务可能会进行回滚，这样这个数据就是无效的。

数据库的TRANSACTIONREADCOMMITTED，TRANSACTIONREPEATABLEREAD，和TRANSACTION_SERIALIZABLE隔离级别可以防止脏读。

### 什么是两阶段提交？
当我们在分布式系统上同时使用多个数据库时，这时候我们就需要用到两阶段提交协议。两阶段提交协议能保证是分布式系统提交的原子性。在第一个阶段，事务管理器发所有的事务参与者发送提交的请求。如果所有的参与者都返回OK，它会向参与者正式提交该事务。如果有任何一个参与方返回了中止消息，事务管理器会回滚所有的修改动作。

### JDBC中存在哪些不同类型的锁？
从广义上讲，有两种锁机制来防止多个用户同时操作引起的数据损坏。

乐观锁——只有当更新数据的时候才会锁定记录。 悲观锁——从查询到更新和提交整个过程都会对数据记录进行加锁。

不仅如此，一些数据库系统还提供了行锁，表锁等锁机制。

### DDL和DML语句分别代表什么？
DDL（数据定义语言，Data Definition Language）语句用来定义数据库模式。Create，Alter, Drop, Truncate, Rename都属于DDL语句，一般来说，它们是不返回结果的。

DML（数据操作语言，Data Manipulation Language）语句用来操作数据库中的数据。select, insert, update, delete, call等，都属于DML语句。

### java.util.Date和java.sql.Date有什么区别？
java.util.Date包含日期和时间，而java.sql.Date只包含日期信息，而没有具体的时间信息。如果你想把时间信息存储在数据库里，可以考虑使用Timestamp或者DateTime字段。

### 如何把图片或者原始数据插入到数据库中？
可以使用BLOB类型将图片或者原始的二进制数据存储到数据库里。

### 什么是幻读，哪种隔离级别可以防止幻读？
幻读是指一个事务多次执行一条查询返回的却是不同的值。假设一个事务正根据某个条件进行数据查询，然后另一个事务插入了一行满足这个查询条件的数据。之后这个事务再次执行了这条查询，返回的结果集中会包含刚插入的那条新数据。这行新数据被称为幻行，而这种现象就叫做幻读。

只有TRANSACTION_SERIALIZABLE隔离级别才能防止产生幻读。

### SQLWarning是什么，在程序中如何获取SQLWarning？
SQLWarning是SQLException的子类，通过Connection, Statement, Result的getWarnings方法都可以获取到它。 SQLWarning不会中断查询语句的执行，只是用来提示用户存在相关的警告信息。

### 如果Oracle的存储过程的入参出参中包含数据库对象，应该如何进行调用？
如果Oracle的存储过程的入参出参中包含数据库对象，我们需要在程序创建一个同样大小的对象数组，然后用它来生成Oracle的STRUCT对象。然后可以通过数据库对象的setSTRUCT方法传入这个struct对象，并对它进行使用。

### 如果java.sql.SQLException: No suitable driver found该怎么办？
如果你的SQL URL串格式不正确的话，就会抛出这样的异常。不管是使用DriverManager还是JNDI数据源来创建连接都有可能抛出这种异常。它的异常栈看起来会像下面这样。
```java
org.apache.tomcat.dbcp.dbcp.SQLNestedException: Cannot create JDBC driver of class 'com.mysql.jdbc.Driver' for connect URL ''jdbc:mysql://localhost:3306/UserDB'
    at org.apache.tomcat.dbcp.dbcp.BasicDataSource.createConnectionFactory(BasicDataSource.java:1452)
    at org.apache.tomcat.dbcp.dbcp.BasicDataSource.createDataSource(BasicDataSource.java:1371)
    at org.apache.tomcat.dbcp.dbcp.BasicDataSource.getConnection(BasicDataSource.java:1044)
java.sql.SQLException: No suitable driver found for 'jdbc:mysql://localhost:3306/UserDB
    at java.sql.DriverManager.getConnection(DriverManager.java:604)
    at java.sql.DriverManager.getConnection(DriverManager.java:221)
    at com.journaldev.jdbc.DBConnection.getConnection(DBConnection.java:24)
    at com.journaldev.jdbc.DBConnectionTest.main(DBConnectionTest.java:15)
Exception in thread "main" java.lang.NullPointerException
    at com.journaldev.jdbc.DBConnectionTest.main(DBConnectionTest.java:16)
```
解决这类问题的方法就是，检查下日志文件，像上面的这个日志中，URL串是'jdbc:mysql://localhost:3306/UserDB，只要把它改成jdbc:mysql://localhost:3306/UserDB就好了。

### 什么是JDBC的最佳实践？
下面列举了其中的一些：

- 数据库资源是非常昂贵的，用完了应该尽快关闭它。Connection, Statement, ResultSet等JDBC对象都有close方法，调用它就好了。
- 养成在代码中显式关闭掉ResultSet，Statement，Connection的习惯，如果你用的是连接池的话，连接用完后会放回池里，但是没有关闭的ResultSet和Statement就会造成资源泄漏了。
- 在finally块中关闭资源，保证即便出了异常也能正常关闭。
- 大量类似的查询应当使用批处理完成。
- 尽量使用PreparedStatement而不是Statement，以避免SQL注入，同时还能通过预编译和缓存机制提升执行的效率。
- 如果你要将大量数据读入到ResultSet中，应该合理的设置fetchSize以便提升性能。
- 你用的数据库可能没有支持所有的隔离级别，用之前先仔细确认下。
- 数据库隔离级别越高性能越差，确保你的数据库连接设置的隔离级别是最优的。
- 如果在WEB程序中创建数据库连接，最好通过JNDI使用JDBC的数据源，这样可以对连接进行重用。
- 如果你需要长时间对ResultSet进行操作的话，尽量使用离线的RowSet。

### 事务的ACID是指什么？ 

- 原子性(Atomic)：事务中各项操作，要么全做要么全不做，任何一项操作的失败都会导致整个事务的失败； 
- 一致性(Consistent)：事务结束后系统状态是一致的； 
- 隔离性(Isolated)：并发执行的事务彼此无法看到对方的中间状态； 
- 持久性(Durable)：事务完成后所做的改动都会被持久化，即使发生灾难性的失败。通过日志和同步备份可以在故障发生后重建数据。

脏读（Dirty Read）：A事务读取B事务尚未提交的数据并在此基础上操作，而B事务执行回滚，那么A读取到的数据就是脏数据。


| 时间	 | 转账事务A| 	取款事务B| 
| --- | --- | --- |
| T1|	 | 	开始事务 | 
| T2| 	开始事务	 | | 
| T3 |	|  	查询账户余额为1000元| 
|T4	 |	|取出500元余额修改为500元|
|T5	|查询账户余额为500元（脏读）|	 |
|T6	| |	撤销事务余额恢复为1000元|
|T7	|汇入100元把余额修改为600元	 ||
|T8	|提交事务	 ||

不可重复读（Unrepeatable Read）：事务A重新读取前面读取过的数据，发现该数据已经被另一个已提交的事务B修改过了。

| 时间| 	转账事务A	| 取款事务B| 
| --- | --- | --- |
| T1| 	 | 	开始事务| 
| T2	| 开始事务	 | | 
| T3	 | 	| 查询账户余额为1000元| 
| T4	| 查询账户余额为1000元	 | | 
| T5| 	 | 	取出100元修改余额为900元| 
| T6| 	 | 	提交事务| 
| T7| 	查询账户余额为900元（不可重复读）	 | | 
幻读（Phantom Read）：事务A重新执行一个查询，返回一系列符合查询条件的行，发现其中插入了被事务B提交的行。

|时间	|统计金额事务A	|转账事务B|
| --- | --- | --- |
|T1	 |	|开始事务|
|T2	|开始事务	 |
|T3	|统计总存款为10000元	 |
|T4	 |	|新增一个存款账户存入100元|
|T5	 |	|提交事务|
|T6	|再次统计总存款为10100元（幻读）	 |
第1类丢失更新：事务A撤销时，把已经提交的事务B的更新数据覆盖了。

|时间|	取款事务A|	转账事务B|
| --- | --- | --- |
|T1	|开始事务	| |
|T2	 ||	开始事务|
|T3	|查询账户余额为1000元	 ||
|T4	|| 	查询账户余额为1000元|
|T5	||	汇入100元修改余额为1100元|
|T6	| |	提交事务|
|T7	|取出100元将余额修改为900元	 ||
|T8	|撤销事务	 ||
|T9	|余额恢复为1000元（丢失更新）|	 |
第2类丢失更新：事务A覆盖事务B已经提交的数据，造成事务B所做的操作丢失。

|时间|	转账事务A	|取款事务B|
| --- | --- | --- |
|T1	 ||	|开始事务|
|T2	|开始事务	| |
|T3	 ||	查询账户余额为1000元|
|T4	|查询账户余额为1000元	| |
|T5	 ||	取出100元将余额修改为900元|
|T6	 ||	提交事务|
|T7	|汇入100元将余额修改为1100元	 ||
|T8	|提交事务	 ||
|T9	|查询账户余额为1100元（丢失更新）|	 |
数据并发访问所产生的问题，在有些场景下可能是允许的，但是有些场景下可能就是致命的，数据库通常会通过锁机制来解决数据并发访问问题，按锁定对象不同可以分为表级锁和行级锁；按并发事务锁定关系可以分为共享锁和独占锁，具体的内容大家可以自行查阅资料进行了解。 
直接使用锁是非常麻烦的，为此数据库为用户提供了自动锁机制，只要用户指定会话的事务隔离级别，数据库就会通过分析SQL语句然后为事务访问的资源加上合适的锁，此外，数据库还会维护这些锁通过各种手段提高系统的性能，这些对用户来说都是透明的（就是说你不用理解，事实上我确实也不知道）。ANSI/ISO SQL 92标准定义了4个等级的事务隔离级别，如下表所示：

| 隔离级别	| 脏读| 	不可重复读| 	幻读|  第一类丢失更新| 	第二类丢失更新| 
| --- | --- | --- | --- | --- | --- |
|READ UNCOMMITED|	允许	|允许	|允许|	不允许|	允许|
|READ COMMITTED|	不允许|	允许|	允许|	不允许|	允许|
|REPEATABLE READ|	不允许|	不允许|	允许|	不允许|	不允许|
|SERIALIZABLE|	不允许|	不允许|	不允许|	不允许|	不允许|
需要说明的是，事务隔离级别和数据访问的并发性是对立的，事务隔离级别越高并发性就越差。所以要根据具体的应用来确定合适的事务隔离级别，这个地方没有万能的原则。

### JDBC中如何进行事务处理？ 
Connection提供了事务处理的方法，通过调用setAutoCommit(false)可以设置手动提交事务；当事务完成后用commit()显式提交事务；如果在事务处理过程中发生异常则通过rollback()进行事务回滚。
除此之外，从JDBC 3.0中还引入了Savepoint（保存点）的概念，允许通过代码设置保存点并让事务回滚到指定的保存点。 
![image_1bmf383no15vdk7e10gd941bkl9.png-4.3kB][2]

### 结束

参考：
[JDBC常见面试题集锦(一)][3]
[JDBC常见面试题集锦(二)][4]


  [1]: http://static.zybuluo.com/homiss/muknb0372s500un09297nb08/image_1blq868ecc3bguks6o17ro1eo919.png
  [2]: http://static.zybuluo.com/homiss/t7cotcyknxrfr0b8yatszdc5/image_1bmf383no15vdk7e10gd941bkl9.png
  [3]: http://it.deepinmind.com/jdbc/2014/03/18/JDBC-common-question-1.html
  [4]: http://it.deepinmind.com/jdbc/2014/03/19/JDBC-common-questions-for-interview.html