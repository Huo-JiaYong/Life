## 个人认识

MongoDB 一种非结构化的数据库

在应对不具有严格性的数据结构的情况下有优异的表现

## 软件安装

1. Windows
   1. 到 mongodb.org 下载 [点击下载](https://www.mongodb.com/download-center?jmp=nav#community)，安装
   2. 使用 mongod --dbpath d:\xxx\data\db 启动服务端
   3. 使用 mongo 启动客户端
2. Linux

## 基础操作

1. 数据库

   1. 创建数据库 

      use dbname [ 也是选择数据库，若数据库不存在则新建 ]

   2. 删除数据库

      db.dropDatabase()

   3. 查看数据库

      show dbs

2. 集合 [ Document 相当于表的概念 ]

   1. 新增集合

      db.createCollection('name', options)

      db.document.insert({xxx:xx}) [ 若 document 不存在则自动新建 ]

   2. 删除集合

      db.dropCollection('name')

3. 数据操作

   **所有的数据都会通过 _id 字段来标识，若不指定则系统自动生成类型为： objectId 的数据**

   1. 新增

      db.document.insert({xx:xx})

      db.document.save({xx:xx})

      db.document.insertOne({xx:xx})

      db.document.insertMany({xx:xx})

   2. 删除

      db.document.remove({xx:xx})

   3. 修改

      db.document.update({xx:xx}[caiteria], {$set:[{xx:xx}], {multi:true})

      caiteria： 标识条件，可以是任何表达式。$and ....

      $set： 表示只修改指定的值，若不指定则会将原对象直接设置为新对象

        eg：原 {name:'xx',age:10} 新 {name:'xx'} 原对象中的 age 也会被清除

      multi：表示修改多行数据，默认只修改一行数据

   4. 查找

      db.document.find({$and:[{$or:[{xx:xx}]}])

        $ne 不等于 

        $lte 小于等于

      .pretty() 表示格式化输出

      .limit(num) 选取几条数据... 从 1 开始

      .skip(num) 跳过多少条数据，即从 n+1 开始获取数据

      .sort({xx:1/-1}) 1 表示升序 -1 表示降序

      **嵌套数据可以通过 xx.xx 的方式设置条件，如：{style:{a:x,b:x,c:x}} style.a 方式指定条件**

   5. 选择字段 [ 只显示部分字段 ]

      db.document.find({},{_id:0,name:1})

      条件后表示字段的是否显示

      _id 默认是显示的，不设置为 0 都会显示

   6. 聚合函数 aggregate()

      db.document.aggregate({$group:[{_id:"$field",num:{$sum:1}]})

      相当于：SELECT field,sum(*) AS num FROM document GROUP BY field

      常用聚合函数都能用 sum avg max min first last

## 在 Spring-boot 中使用

1. 引入 jar

   spring-boot-starter-data-mongodb

2. 添加配置

   data:mongodb:database:

3. 使用 MongoTemplate

   template.save(Object, "document") 参数：对象和集合名

   template.findAll(query, xx.Class, "document") 查询出数组

   template.update(query, update, xx.Class, "document")

   template.remove(query, xx.Class, "document")