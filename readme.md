# 淘宝双十一数据分析与预测

# 一、本地数据上传到数据仓库 hive

## 1.1 实验数据 

## 1.1.1、用户行为日志 user_log.csv

日志中字段定义如下：

- user_id 买家id
- item_id 商品id
- cat_id 商品类别id
- merchant_id 卖家id
- brand_id 品牌id
- month 交易时间:月
- day 交易事件:日
- action 行为,取值范围{0,1,2,3},
	- 0表示点击
	- 1表示加入购物车
	- 2表示购买
	- 3表示关注商品
- age_range 买家年龄分段
	- 1表示年龄<18
	- 2表示年龄在[18,24]
	- 3表示年龄在[25,29]
	- 4表示年龄在[30,34]
	- 5表示年龄在[35,39]
	- 6表示年龄在[40,49]
	- 7和8表示年龄>=50
	- 0和NULL则表示未知
- gender 性别
	- 0表示女性
	- 1表示男性
	- 2和NULL表示未知
- province 收获地址省份

## 1.2 本地数据上传到数据仓库Hive

> 上传测试数据到HDFS

```
# hdfs dfs -mkidr -p /taobao/input
# hdfs dfs -put user_log.csv  /taobao/input
```

> Hive中创建数据库

```
hive (default)> create database taobao;
```
> 切换数据库

```
hive (default)> use taobao;
```
> 创建外部表

```
hive (taobao)>  create external table if not exists user_log(user_id int ,item_id int,cat_id int, merchant_id int,brand_id int,month string,day string,action int,age_range int,gender int,province string) row format delimited fields terminated by "," location  '/taobao/input/';
```
> 查看数据

```
hive (taobao)> select * from user_log limit 5;
```
# 二、使用Sqoop将数据从Hive导入MySQL

## 2.1、创建临时表inner_user_log

```
create table taobao.inner_user_log(user_id INT,item_id INT,cat_id INT,merchant_id INT,brand_id INT,month STRING,day STRING,action INT,age_range INT,gender INT,province STRING)  ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' ;
```
## 2.2、 导入数据

```
INSERT OVERWRITE TABLE taobao.inner_user_log select * from taobao.user_log;
```

## 2.3、MySQL创建数据库

```
CREATE DATABASE `taobao` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci */
```
## 2.4、 MySQL创建数据表

```
CREATE TABLE `user_log` (
  `user_id` varchar(20) DEFAULT NULL,
  `item_id` varchar(20) DEFAULT NULL,
  `cat_id` varchar(20) DEFAULT NULL,
  `merchant_id` varchar(20) DEFAULT NULL,
  `brand_id` varchar(20) DEFAULT NULL,
  `month` varchar(6) DEFAULT NULL,
  `day` varchar(6) DEFAULT NULL,
  `action` varchar(6) DEFAULT NULL,
  `age_range` varchar(6) DEFAULT NULL,
  `gender` varchar(6) DEFAULT NULL,
  `province` varchar(10) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
## 2.5、sqoop 将hive中的数据导入mysql

```
# sqoop export --connect jdbc:mysql://localhost:3306/taobao --username root -P --table user_log --export-dir '/user/hive/warehouse/taobao.db/inner_user_log' --fields-terminated-by ',';
```
> 字段解析

-  sqoop export ##表示数据从hive复制到mysql中
- –connect jdbc:mysql://localhost:3306/dbtaobao
- –username root #mysql登陆用户名
- –P   #登录密码
- –table user_log #mysql 中的表，即将被导入的表名称
- –export-dir ‘/user/hive/warehouse/dbtaobao.db/user_log ‘ #hive 中被导出的文件
- –fields-terminated-by ‘,’ #Hive 中被导出的文件字段的分隔符