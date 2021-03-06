---
date: "2019-06-26T23:14:48+08:00"
draft: false
title: "mysql事务隔离级别"
tags: ["Mysql"]
series: ["mysql"]
categories: ["Mysql"]
toc: true
---

## 隔离性
提到事务，我们都知道ACID（Atomicity、Consistency、Isolation、Durability，即原子性、一致性、隔离性、持久性），
今天我们就来说说其中I，也就是“隔离性”。

当数据库上有多个事务同时执行的时候，就可能出现脏读（dirty read）、不可重复读（non-repeatable read）、
幻读（phantom read）的问题。为了解决这些问题，我们就需要引入"隔离性"的概念，从而就有了“隔离级别”。

## 隔离级别
在谈隔离级别之前，首先我们要知道，隔离得越严实，效率就会越低。因此很多时候，我们都要在二者之间寻找一个平衡点。

标准的事务隔离级别包括：读未提交（read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（serializable ）。

>`读未提交`是指，一个事务还没提交时，它做的变更就能被别的事务看到。     
`读提交`是指，一个事务提交之后，它做的变更才会被其他事务看到。      
`可重复读`是指，一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。      
`串行化`，顾名思义是对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

下面我们用一个例子来说明：
```sql
mysql> create table test(id int) engine=InnoDB;
insert into test(id) values(1);
```
首先我们新建一张test表，里面只有id这个int类型的字段。
{{% center %}}<img name="touchbar-config" src="/images/blog/2019-06/mysql_04.png" width='200px'/>{{% /center %}}

若隔离级别是“读未提交”，则V1的值就是2。这时候事务B虽然还没有提交，但是结果已经被A看到了。因此，V2、V3也都是2。 

若隔离级别是“读提交”，则V1是1，V2的值是2。事务B的更新在提交后才能被A看到。所以，V3的值也是2。 

若隔离级别是“可重复读”，则V1、V2是1，V3是2。之所以V2还是1，遵循的就是这个要求：事务在执行期间看到的数据前后必须是一致的。 

若隔离级别是“串行化”，则在事务B执行“将1改成 2”的时候，会被锁住。直到事务A提交后，事务B才可以继续执行。
所以从A的角度看，V1、V2值是1，V3的值是2。

