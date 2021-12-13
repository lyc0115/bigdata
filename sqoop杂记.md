<h1 align = "center">sqoop杂记</h1>

# 一、`sqoop`介绍

​	`Sqoop`是一款开源的工具，主要用于在`Hadoop(Hive、Hbase)`与传统的数据库(`mysql、postgresql...`)间进行数据的传递，可以将一个关系型数据库（例如 ：` MySQL ,Oracle ,Postgres`等）中的数据导进到`Hadoop`的`HDFS`中，也可以将`HDFS`的数据导进到关系型数据库中。以下操作环境为`CDH5.14`

# 二、`sqoop`导入数据

​	在`Sqoop`中，“导入”概念指：从非大数据集群（`RDBMS`）向大数据集群（`HDFS，HIVE，HBASE`）中传输数据，使用import关键字

## 2.1 `RDBMS`到`HDFS`

### 2.1.1 全部导入

![code-snapshot](https://gitee.com/GaborPca/document/raw/master/img1/code-snapshot.png)

### 2.1.2 查询导入

![code-snapshot (https://gitee.com/GaborPca/document/raw/master/img1/code-snapshot2.png)](C:\Users\MECHREVO\Downloads\code-snapshot2.png)

> <font color='red'>''$CONDITIONS"必须包含在where字句中 </font>

### 2.1.3 导入指定列

![code-snapshot4](https://gitee.com/GaborPca/document/raw/master/img1/code-snapshot4.png)

## 2.2 `RDBMS`到`HBASE`

![carbon](https://gitee.com/GaborPca/document/raw/master/img1/carbon.png)

> <font color='red'>手动在hbase中创建迁移表表名</font>

```sql
create 'pf_achievement_award', 'info'
```

## 2.3 `RDBMS`到`HIVE`

+ 将关系型数据库数据导入到`hbase`中
+ `hive`中建立`habse`的外部表，对表数据进行操作

# 三、`sqoop`导出数据

​	在`sqoop`中，“导出”概念指：从大数据集群（`HDFS，HIVE，HBASE`）向非大数据集群（`RDBMS`）中传输数据，使用export关键字

## 3.1 `HIVE/HDFS`到`RDBMS`

![code-snapshot5](https://gitee.com/GaborPca/document/raw/master/img1/code-snapshot5.png)

> <font color='red'>mysql提前创建表perosn</font>

# 四、增量导入数据

## 4.1 `MYSQL`到`HBASE`

使用`sqoop`自带的`job`机制，生成`sqoop`任务，不会立即执行

```shell
# 查询当前job任务
sqoop job --list
# 显示指定job任务详情
sqoop job --show t_emp
# 删除job任务
sqoop job --delete t_emp
# 执行job任务
sqoop job --exec t_emp
```

+ 根据指定id主键进行判断增量导入

  ![code-snapshot6](https://gitee.com/GaborPca/document/raw/master/img1/code-snapshot6.png)

> 为了避免每次输入密码，上述脚本采用`--password-file`参数，将密码保存至`sqoopPWD.pwd`中，操作如下

```shell
touch sqoopPWD.pwd
echo -n 'root' > sqoopPWD.pwd
hadoop fs -put sqoopPWD.pwd /sqoop/pwd
hadoop fs -chmod 400 /sqoop/pwd/sqoopPWD.pwd
```

编写执行脚本`sqoop-exec.sh`

```shell
#!/bin/bash
sqoop job --exec t_emp
```

通过配合`linux`中`crontab`定时任务，定期对`mysql`数据进行导入 

```shell
# 每天凌晨5点执行一次
0 5 * * * sqoop-exec.sh
```

