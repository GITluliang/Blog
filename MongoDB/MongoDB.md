

### 基本概念：

* 数据库（database）、集合（collection）、文档（document）
* 数据库里存集合，集合里存文档；

* 在MongoDB中，数据库和集合都不需要我们手动创建；当创建文档时，如果数据库和集合不存在会自动创建数据库和集合；

### 基本指令：

* show databases：显示所有数据库

* use 数据库名：进入指定的数据库中

* db：查看当前数据库

* show collections：显示数据库中所有集合

* 数据库的CRUD：

  * db.<collection>.insert(doc)：向集合中插入一个文档

    ```
    db.stus.insert({nama:"孙悟空",age:18,gender:"男"})
    ```

  * db.<collection>.find()：查询当前集合中的所有文档

    ```
    db.stus.find()
    ```

