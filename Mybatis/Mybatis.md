# Mybatis



#### **1.mybatis中的#和$的区别：**

1. #将传入的数据都当成一个字符串，会对自动传入的数据加一个双引号。
   如：where username=#{username}，如果传入的值是111,那么解析成sql时的值为where username="111", 如果传入的值是id，则解析成的sql为where username="id".　
2. $将传入的数据直接显示生成在sql中。
   如：where username=${username}，如果传入的值是111,那么解析成sql时的值为where username=111；
   如果传入的值是;drop table user;，则解析成的sql为：select id, username, password, role from user where username=;drop table user;
3. #方式能够很大程度防止sql注入，$方式无法防止Sql注入。
4. $方式一般用于传入数据库对象，例如传入表名.
5. 一般能用#的就别用$，若不得不使用“${xxx}”这样的参数，要手工地做好过滤工作，来防止sql注入攻击。
6. 在MyBatis中，“${xxx}”这样格式的参数会直接参与SQL编译，从而不能避免注入攻击。但涉及到动态表名和列名时，只能使用“${xxx}”这样的参数格式。所以，这样的参数需要我们在代码中手工进行处理来防止注入。

**【结论】在编写MyBatis的映射语句时，尽量采用“#{xxx}”这样的格式。若不得不使用“${xxx}”这样的参数，要手工地做好过滤工作，来防止SQL注入攻击。**

原文：[SQL注入](https://www.cnblogs.com/myseries/p/10821372.html)



#### 2.mybatis是如何做到防止sql注入的

MyBatis框架作为一款半自动化的持久层框架，其SQL语句都要我们自己手动编写，这个时候当然需要防止SQL注入。其实，MyBatis的SQL是一个具有“**输入+输出**”的功能，类似于函数的结构。其中，parameterType表示了输入的参数类型，resultType表示了输出的参数类型。回应上文，如果我们想防止SQL注入，理所当然地要在输入参数上下功夫。代码中使用#的即输入参数在SQL中拼接的部分，传入参数后，打印出执行的SQL语句，会看到SQL是这样的：

```SQL
select id, username, password, role from user where username=? and password=?
```

不管输入什么参数，打印出的SQL都是这样的。这是因为MyBatis启用了预编译功能，在SQL执行前，会先将上面的SQL发送给数据库进行编译；执行时，直接使用编译好的SQL，替换占位符“?”就可以了。因为SQL注入只能对编译过程起作用，所以这样的方式就很好地避免了SQL注入的问题。

【底层实现原理】MyBatis是如何做到SQL预编译的呢？其实在框架底层，是JDBC中的PreparedStatement类在起作用，PreparedStatement是我们很熟悉的Statement的子类，它的对象包含了编译好的SQL语句。这种“准备好”的方式不仅能提高安全性，而且在多次执行同一个SQL时，能够提高效率。原因是SQL已编译好，再次执行时无需再编译