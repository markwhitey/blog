---
title:  ScalaLikeJDBC（基于SQL的简洁DB访问）
mathjax: false
date: 2019-09-28 14:30:18
tags:
- Big Data
- Scala
categories:
- Big Data
- Scala
top:
photo:
---



{% cq %}

ScalikeJDBC是一款给Scala开发者使用的简洁DB访问类库，它是基于SQL的，使用者只需要关注SQL逻辑的编写，所有的数据库操作都交给ScalikeJDBC。这个类库内置包含了JDBC API，并且给用户提供了简单易用并且非常灵活的API。并且，QueryDSL(通用查询查询框架)使你的代码类型安全的并且可重复使用。我们可以在生产环境大胆地使用这款DB访问类库

{% endcq %}

<!-- more -->

<br>





# 概念

​		ScalikeJDBC是一款给Scala开发者使用的简洁DB访问类库，它是基于SQL的，使用者只需要关注SQL逻辑的编写，所有的数据库操作都交给ScalikeJDBC。这个类库内置包含了JDBC API，并且给用户提供了简单易用并且非常灵活的API。并且，QueryDSL(通用查询查询框架)使你的代码类型安全的并且可重复使用。我们可以在生产环境大胆地使用这款DB访问类库。

# 官网

http://scalikejdbc.org/

# 项目中使用ScalaLikeJDBC

## Step1=>>在pom.xml导入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.scalikejdbc</groupId>
        <artifactId>scalikejdbc_2.11</artifactId>
        <version>3.1.0</version>
    </dependency>
    <dependency>
        <groupId>org.scalikejdbc</groupId>
        <artifactId>scalikejdbc-config_2.11</artifactId>
        <version>3.1.0</version>
    </dependency>
    <dependency>
    	<groupId>mysql</groupId>
    	<artifactId>mysql-connector-java</artifactId>
    	<version>5.1.38</version>
	</dependency>
</dependencies>
```

## Step2=>>添加配置文件

​		resources文件夹中创建**==application.conf==**，**文件名固定不可更改（这样在第三步中，才可以使用setup导入配置文件，否则需要使用setupAll来导入）**。

```scala
// 默认的配置就是db.default.xxx，如果使用其他配置需要setup('name)指明
db.default.driver="com.mysql.jdbc.Driver"
db.default.url="jdbc:mysql://localhost：3306/databasename?characterEncoding=utf-8"
db.default.user="username"
db.default.password="password"
 
// 连接池配置格式如下，具体属性查官网
// 连接池初始大小
// db.default.poolInitialSize=100
```

## Step3=>>数据库CRUD

### part1=>>数据库基础CRUD

```scala
import scalikejdbc.{DB, SQL}
import scalikejdbc.config.DBs

object ScaLikeJDBCDemo1 {
  def main(args: Array[String]): Unit = {
    //scalikejdbc不需要人为的导入连接，也不需要人为的关闭连接
    //scalikejdbc操作之前必须得有表
    // 加载配置
    DBs.setup()
    Create
    Delete
    Update
    Retrieve
  }

  // 插入数据
  def Create = {
    DB.localTx(implicit session=>{
      // SQL里面是普通的sql语句
      SQL("insert into ADCarry values (?,?,?,?)")
      // bind()传参
        .bind(1,"VN",23,"famale")
      // 启动执行命令
        .update()
      // 发送请求
        .apply()
    })
    println("===>>>执行结束")
  }

  // 删除数据
  def Delete = {
    DB.autoCommit(implicit session=>{
      SQL("delete from ADCarry where name = ?").bind("EZ").update().apply()
    })
    println("===>>>执行结束")
  }

  // 修改数据
  def Update = {
    DB.autoCommit(implicit session=>{
      SQL("update ADCarry set age=? where name=?").bind(18,"EZ").update().apply()
    })
    println("===>>>执行结束")
  }

  // 查询数据
  def Retrieve = {
    val tuples = DB.readOnly(implicit session => {
      SQL("select * from ADCarry").map(line => (
        line.int("id"),
        line.string("name"),
        line.int("age"),
        line.string("gender")
      )).list().apply()
    })
    tuples.foreach(print(_))
  }
```

### part2=>>插入多条数据及事务

```scala
def main(args: Array[String]): Unit = {
    DBs.setup()
    Some
}

  // 插入多条数据以及事务
  def Some = {
    val tuples = Array((4,"Draven",35,"male"),(5,"EZ",18,"male"))
    DB.localTx{implicit session=>
       tuples.foreach(tp=>{
          SQL("insert into ADCarry values (?,?,?,?)").bind(tp._1,tp._2,tp._3,tp._4).update().apply()
       })
    }
  }
```

### part3=>>Spark写入数据库

导入依赖:

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.11</artifactId>
    <version>2.2.0</version>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.11</artifactId>
    <version>2.2.0</version>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.11</artifactId>
    <version>2.2.0</version>
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka_2.11</artifactId>
    <version>0.10.2.2</version>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-kafka-0-10_2.11</artifactId>
    <version>2.2.0</version>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

代码实现

```scala
  def main(args: Array[String]): Unit = {
    DBs.setup()
    sparkDemo
  }

 // spark写入数据库
  def sparkDemo = {
    val session = SparkSession.builder().master("local[*]").appName(this.getClass.getName).getOrCreate()
    // 创建rdd
    val rddTuples = session.sparkContext.makeRDD(Array((4,"Draven",35,"male"),(5,"EZ",18,"male")))
    rddTuples.foreachPartition(partitions=>{
      DB.localTx{implicit session=>
        partitions.foreach(tuple=>{
          SQL("insert into ADCarry values (?,?,?,?)").bind(tuple._1,tuple._2,tuple._3,tuple._4).update().apply()
        })
      }
    })
  }
```

创建confi类

```scala
import com.typesafe.config.{Config, ConfigFactory}
  
  object confi {
    //加载application.conf配置文件
    private lazy val load: Config = ConfigFactory.load()
    //加载驱动
    val driver: String = load.getString("db.default.driver")
    //加载url
    val url: String = load.getString("db.default.url")
    //加载用户名
    val user: String = load.getString("db.default.user")
    //加载密码
    val password: String = load.getString("db.default.password")
  }
```

代码实现

```scala
  def main(args: Array[String]): Unit = {
    DBs.setup()
    sparkDemo2
  }

  def sparkDemo2 ={
    val session = SparkSession.builder().master("local[*]").appName(this.getClass.getName).getOrCreate()
    import session.implicits._
    val rddTuples = session.sparkContext.makeRDD(Array((4,"Draven",35,"male"),(5,"EZ",18,"male")))
    val props = new Properties()
    props.setProperty("driver",config.driver)
    props.setProperty("user",config.user)
    props.setProperty("password",config.password)
       rddTuples.toDF("id","name","age","gender").write.mode(SaveMode.Append).jdbc(config.url,"ADCarry",props)
    session.stop()
  }
```

