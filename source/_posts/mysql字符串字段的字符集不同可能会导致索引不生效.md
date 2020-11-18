---
title: mysql字符串字段的字符集不同可能会导致索引不生效
date: 2020-11-17 01:37:26
tags: 
    - mysql
categories: 
    - program
---
前言，在探索这个问题之前，我们需要了解 mysql 的一个基本概念。mysql 有三种数据类型，数值型、字符串型、日期型，其中字符串类型需要设置字符集。如果不设置，在创建表结构的时候会默认使用数据库的字符集。
### 1.准备两个测试表（test_a, test_b）。
```
CREATE TABLE `test_a` (
  `id` bigint NOT NULL COMMENT '主键编号',
  `test_bigint` bigint DEFAULT NULL COMMENT 'utf8的bigint',
  `test_varchar_utf8` varchar(255) DEFAULT NULL COMMENT 'utf8的varchar',
  `test_varchar_utf8mb4` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT 'utf8mb4的varchar',
  PRIMARY KEY (`id`),
  KEY `idx_test_bigint` (`test_bigint`) USING BTREE COMMENT 'test_bigint索引',
  KEY `idx_test_varchar_utf8` (`test_varchar_utf8`) USING BTREE COMMENT 'test_varchar_utf8索引',
  KEY `idx_test_varchar_utf8mb4` (`test_varchar_utf8mb4`) USING BTREE COMMENT 'test_varchar_utf8mb4索引'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `test_b` (
  `id` bigint NOT NULL COMMENT '主键编号',
  `test_bigint` bigint DEFAULT NULL COMMENT 'utf8的bigint',
  `test_varchar_utf8` varchar(255) DEFAULT NULL COMMENT 'utf8的varchar',
  `test_varchar_utf8mb4` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT 'utf8的varchar',
  PRIMARY KEY (`id`),
  KEY `idx_test_bigint` (`test_bigint`) USING BTREE COMMENT 'test_bigint索引',
  KEY `idx_test_varchar_utf8` (`test_varchar_utf8`) USING BTREE COMMENT 'test_varchar_utf8索引',
  KEY `idx_test_varchar_utf8mb4` (`test_varchar_utf8mb4`) USING BTREE COMMENT 'test_varchar_utf8mb4索引'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
**注意：这两个表的 test_varchar_utf8 字段使用的是utf8字符集，test_varchar_utf8mb4 字段使用的是 utf8mb4 字符集。mysql 的 utf8 使用最大3字节长度来编码字符，而 utf8mb4 是使用最大4字节长度编码字符，可以理解为 utf8mb4 是 utf8 的超集。**
### 2.插入几条测试数据
在 test_a 表中插入一条数据，在 test_b 表中插入10条数据。多少条数据无所谓，只需要能达到测试效果即可。
```
INSERT INTO `test_a`(`id`, `test_bigint`, `test_varchar_utf8`, `test_varchar_utf8mb4`) VALUES (1, 1, '1', '1');

INSERT INTO `test_b`(`id`, `test_bigint`, `test_varchar_utf8`, `test_varchar_utf8mb4`) VALUES (1, 1, '1', '1');
INSERT INTO `test_b`(`id`, `test_bigint`, `test_varchar_utf8`, `test_varchar_utf8mb4`) VALUES (2, 2, '2', '2');
INSERT INTO `test_b`(`id`, `test_bigint`, `test_varchar_utf8`, `test_varchar_utf8mb4`) VALUES (3, 3, '3', '3');
INSERT INTO `test_b`(`id`, `test_bigint`, `test_varchar_utf8`, `test_varchar_utf8mb4`) VALUES (4, 4, '4', '4');
INSERT INTO `test_b`(`id`, `test_bigint`, `test_varchar_utf8`, `test_varchar_utf8mb4`) VALUES (5, 5, '5', '5');
INSERT INTO `test_b`(`id`, `test_bigint`, `test_varchar_utf8`, `test_varchar_utf8mb4`) VALUES (6, 6, '6', '6');
INSERT INTO `test_b`(`id`, `test_bigint`, `test_varchar_utf8`, `test_varchar_utf8mb4`) VALUES (7, 7, '7', '7');
INSERT INTO `test_b`(`id`, `test_bigint`, `test_varchar_utf8`, `test_varchar_utf8mb4`) VALUES (8, 8, '8', '8');
INSERT INTO `test_b`(`id`, `test_bigint`, `test_varchar_utf8`, `test_varchar_utf8mb4`) VALUES (9, 9, '9', '9');
INSERT INTO `test_b`(`id`, `test_bigint`, `test_varchar_utf8`, `test_varchar_utf8mb4`) VALUES (10, 10, '10', '10');
```

### 3.准备测试的 sql 语句，并执行
通过 test_a 表的数据关联查询 test_b 表的数据。
```
-- 测试非字符串类型的查询
EXPLAIN SELECT * FROM test_a ta INNER JOIN test_b tb ON tb.test_bigint = ta.test_bigint WHERE ta.test_bigint = 1;
-- 测试字符串类型，使用utf8编码的字段查询utf8编码的字段
EXPLAIN SELECT * FROM test_a ta INNER JOIN test_b tb ON tb.test_varchar_utf8 = ta.test_varchar_utf8 WHERE ta.test_varchar_utf8 = '1';
-- 测试字符串类型，使用utf8mb4编码的字段查询utf8编码的字段
EXPLAIN SELECT * FROM test_a ta INNER JOIN test_b tb ON tb.test_varchar_utf8 = ta.test_varchar_utf8mb4 WHERE ta.test_varchar_utf8 = '1';
-- 测试字符串类型，使用utf8编码的字段查询utf8mb4编码的字段
EXPLAIN SELECT * FROM test_a ta INNER JOIN test_b tb ON tb.test_varchar_utf8mb4 = ta.test_varchar_utf8 WHERE ta.test_varchar_utf8 = '1';
-- 测试字符串类型，使用utf8mb4编码的字段查询utf8mb4编码的字段
EXPLAIN SELECT * FROM test_a ta INNER JOIN test_b tb ON tb.test_varchar_utf8mb4 = ta.test_varchar_utf8mb4 WHERE ta.test_varchar_utf8 = '1';
```
以下是执行结果

**结果1：**
![图 3-1.png](https://blog-rookie0peng.oss-cn-shenzhen.aliyuncs.com/github/pages/2020/1119/d5d1c78846aa4832882f2e75b5523f2e.png)
**结果2：**
![图 3-2.png](https://blog-rookie0peng.oss-cn-shenzhen.aliyuncs.com/github/pages/2020/1119/8ce85144b90144a3b63cf31b33281692.png)
**结果3：**
![图 3-3.png](https://blog-rookie0peng.oss-cn-shenzhen.aliyuncs.com/github/pages/2020/1119/f70177102e47452c93de9664b78059aa.png)
**结果4：**
![图 3-4.png](https://blog-rookie0peng.oss-cn-shenzhen.aliyuncs.com/github/pages/2020/1119/9638030ad8374fd8b8e62c7cbb062b2a.png)
**结果5：**
![图 3-5.png](https://blog-rookie0peng.oss-cn-shenzhen.aliyuncs.com/github/pages/2020/1119/d07ae072b3814f7b9ae80163779130f9.png)

### 4.分析执行结果
##### 4-1.对于 table ta 的查询始终是走索引的
观察5个结果中 table ta 的 key 字段，都是有值的。这说明了在使用 where 条件显式查询时，不管目标字段的字符集是什么，都会在查询的过程中默认使用该字符集进行编译，然后就可以走索引。
##### 4-2.对于 table tb 的查询，会因为字符集的不同导致不走索引
观察5个结果中的 table tb 的 key 字段，只有结果3的 key 字段为 null，再看看结果3的查询语句，使用 utf8mb4 编码的字段查询 utf8 编码的字段，联想开篇提到的内容，utf8mb4 是 utf8 的超集，那我们使用 utf8mb4 编码的字段去查询 utf8 编码的字段会出现不兼容的情况，没法走索引。
##### 4-3. 对于 table tb 的查询，如果目标字段的字符集可以兼容源字段的字符集，还是可以走索引。
观察第4条 sql 语句，使用 utf8 编码的字段查询 utf8mb4 编码的字段，再看它的执行结果，table tb的 key 字段是有值的，代表走了索引，联想开篇提到的内容，utf8mb4 是 utf8 的超集，那么就可以理解为啥字符集不同也可以走索引了。
### 5.结论
mysql 字符串字段的字符集不同可能会导致索引不生效
### 6.建议
为了方便使用，还是统一数据库字符集比较好。
如果没法修改字段的字符集的话，也可以使用强转字符集的方式，让字段走索引。我们将第3条sql语句按照下面的方式进行调整。
##### 6-1.将 utf8 强转成 utf8mb4，不走索引
将 utf8 强转成 utf8mb4 时，需要设置字符集的排序规则
```
EXPLAIN SELECT * FROM test_a ta INNER JOIN test_b tb ON CONVERT(tb.test_varchar_utf8 USING utf8mb4) COLLATE utf8mb4_general_ci = ta.test_varchar_utf8mb4 WHERE ta.test_varchar_utf8mb4 = '1';
```
**执行结果如下：**
![图 6-1.png.png](https://blog-rookie0peng.oss-cn-shenzhen.aliyuncs.com/github/pages/2020/1119/7c6fd6a54f3c4024bf636d31e263d1cc.png)
从图 6-1中可以看到，table tb 的 key 字段的值为 null，代表它没走索引。也就是说，将 utf8 强转成 utf8mb4，并不会走索引。
##### 6-2.将 utf8mb4 强转成 utf8，走索引
将 utf8mb4 强转成 utf8 时，不需要设置字符集的排序规则
```
-- 测试字符串类型，通过utf8mb4查询utf8，将utf8mb4强转成utf8
EXPLAIN SELECT * FROM test_a ta INNER JOIN test_b tb ON tb.test_varchar_utf8 = CONVERT(ta.test_varchar_utf8mb4 USING utf8) WHERE ta.test_varchar_utf8mb4 = '1';
```
**执行结果如下：**
![图 6-2.png](https://blog-rookie0peng.oss-cn-shenzhen.aliyuncs.com/github/pages/2020/1119/f4e77a4a665845468e22db0a871209da.png)
从图 6-2中可见，table tb 的 key 字段是有值的，这代表它走了索引，完结撒花！