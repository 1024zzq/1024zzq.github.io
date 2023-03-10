# MySQL索引

> 索引(Index)是帮助MySQL高效获取数据的数据结构。 0.5s    0.000001s
>
> 索引是数据结构

使用`select…for update`给数据加锁的时候，咱们需要注意锁的级别，MySQL InnoDB 默认行级锁。

==行级锁都是基于索引的，如果一条 SQL 语句用不到索引是不会使用行级锁的，而会使用表级锁把整张表锁住，这点需要咱们格外的注意==

要使用行级锁：查询或者更新条件必须是索引字段

# 索引的分类

* 主键索引（PRIMERY KEY)
* 唯一索引（UNIQUE KEY）
  * 避免重复列
* 常规索引（KEY/INDEX）
  * 用index，key关键字来设置
* 全文索引（FullText）
  * 只有在特定的数据库引擎中才有



加了索引之后的效果：

![img](https://gitee.com/codezzq/blogImage/raw/d7b1ce934a79f339f7416ca6fba4981969d003bf/img/image-20201124203324012.png)

注：索引在小数据量的时候用处不大，但是在大数据的时候，区别十分明显。

# 索引原则

* 索引不是越多越好
* 不要对经常变动的数据加索引
* 小数据量的表不需要加索引
* 索引一般加在常用来查询的字段上

> 索引的数据结构

Hash 类型的索引

Btree：InnoDB的默认数据结构



#### 注意自连接

交叉连接（CROSS JOIN）一般用来返回连接表的笛卡尔积。

> 本节的末尾介绍了笛卡尔积，不了解笛卡尔积的读者可以先阅读文章末尾部分，然后再继续学习交叉连接。

交叉连接的语法格式如下：

SELECT <字段名> FROM <表1> CROSS JOIN <表2> [WHERE子句]

或

SELECT <字段名> FROM <表1>, <表2> [WHERE子句] 

==BUT==：要用 cross join 才能用 on，实测用 ，不行！！！

cross join weather w2 on datediff(w1.recordDate,w2.recordDate) = 1







总结，using() 只是join中指定连接条件的简写，在简单的连接中常用。在==列名称不同==时或连接条件复杂时就无法用了，使用 a left join b on ... 是更常见的做法。









# 待完善

> 结合 javaguide & 廖雪峰 & 03、本地事务&分布式事务    后期给个链接   结合45讲笔记 #隔离性与隔离级别

## 隔离级别

READ COMMITTED（读提交）  Oracle 和 SQL Server 的默认隔离级别













# 做 LeetCode 汇总

#### 注意 update 没有 from

```mysql
DELETE FROM 表名
UPDATE 表名 SET 字段名=新值
```



#### 可以把 null 字段过滤成 0    或者用 if

```mysql
select e1.employee_id, IFNULL(e2.salary ,0) bonus
from employees e1 left join
```



#### IF 表达式 IF( expr1 , expr2 , expr3 ) expr1 的值为 TRUE，则返回值为 expr2 expr1 的值为FALSE，则返回值为 expr3

```mysql
update salary set sex = if(sex = 'm','f','m')
```



#### mysql 中调用下面的语句提示该错误

> In MySQL, you can’t modify the same table which you use in the SELECT part.

```mysql
delete from Person where id in (
    select t1.id from Person t1, Person t2 where t1.email = t2.email and t1.id > t2.id
);
```

解决方法:  建立一张临时的中间表

```mysql
delete from Person where id in (
    select id from(
          select t1.id as id from Person t1, Person t2 where t1.email = t2.email and t1.id > t2.id
    ) as tmp
);
```



#### 如果表名用了别名，要将别名放在delete和from之间

```mysql
把重复的用Delete 删除就好了

delete u
from Person u , Person v
where v.id < u.id and u.email = v.email 
```



#### ==用 Group by 删除后面id值大的重复的数据==

```mysql
DELETE from Person 
Where Id not in (
    Select Id 
    From(
    Select MIN(Id) as id
    From Person 
    Group by Email
   ) t
)
```



#### [学习自连接](https://zhuanlan.zhihu.com/p/141083771)：自连接：自己和自己做笛卡尔积



#### 名字首字母大写，后面小写

> JavaGuide 中都搜不到 upper lower 这两个函数

```mysql
concat(upper(left(name,1)),lower(substring(name,2))) name # substr 好像也可以
```



#### group_concat([distinct] 字段名 [order by 排序字段 asc/desc] [separator '分隔符'])

```mysql
group_concat 返回带有来自一个组的连接的非NULL值的字符串结果。组内字符串连接

group_concat(distinct product order by product separator ',') as products
```



#### 列转行用union all：一开始我没那么理解

```mysql
#'store1' store 值store1在store这个字段名（别名）下！
select product_id, 'store1' store, store1 price from products where store1 is not null
union 
select product_id, 'store2' store, store2 price from products where store2 is not null
union
select product_id, 'store3' store, store3 price from products where store3 is not null
```

补充：==UNION去重且排序  UNION ALL不去重不排序==

==union语句注意事项==：没答出来

         1.通过union连接的SQL它们分别单独取出的列数必须相同；
    
         2.不要求合并的表列名称相同时，以第一个sql 表列名为准；
    
         3.使用union 时，完全相等的行，将会被合并，由于合并比较耗时，一般不直接使用 union 进行合并，而是通常采用union all 进行合并；
    
         4.被union 连接的sql 子句，单个子句中不用写order by ，因为不会有排序的效果。但可以对最终的结果集进行排序；

**union会自动将完全重复的数据去除掉，union all会保留那些重复的数据；**



#### case的使用

```mysql
case 
	 when xx then xx
	 when xx then xx
	 ...
	 else xx
end
```

任何与`null`值的对比都将返回`null`. 因此返回结果为否,这点可以用代码 `select if(1 = null, 'true', 'false')`证实.

我在这里犯了两个错误。第一个是常见的not in 和null之间的错误，==任何与null值的比对都会返回null==，因此判断语句就无法生效。

```mysql
select 3 not in (null,2)
```



#### [第二高的薪水](https://leetcode.cn/problems/second-highest-salary/)

```mysql
select (select distinct salary from Employee order by salary desc limit 1,1) as SecondHighestSalary
```



#### DATEDIFF() 函数返回两个日期之间的天数。

&  [具体所有 Date 相关函数](https://www.w3school.com.cn/sql/sql_dates.asp) 建议看这个

#### Date 时间格式的处理函数：取date类型字段为 2021年8月 的数据

```mysql
# 1
SELECT DATEDIFF('2008-12-30','2008-12-29') AS DiffDate
# -1
SELECT DATEDIFF('2008-12-29','2008-12-30') AS DiffDate


#Date学习  简单点：Date可以当 String处理
date between '2021-08-01'and '2021-08-31' #日期也可以between，注意：得带引号不然就错了！！！
year(date)=2021 and month(date)=08
date like '2021-08%'
date regexp '2021-08'
substring(date,1,7) = '2021-08'
#在mysql中可以使用MONTH()函数来获取月份，它会从指定日期值中获取月份值。MONTH()函数需要接受date参数，语法格式为“MONTH(date);”，返回指定date对应的月份，范围为“1～12”。
year(date); MONTH(date); day(date);
```



#### 要创建1天间隔，请使用以下表达式：

```mysql
interval 1 day
# 场景：查询近30天活跃用户数
where activity_date > date_sub('2019-07-27', interval 30 day)
DATE_ADD()
# 其实有更简单的：注意会有负数要 and
where datediff('2019-07-27', activity_date) < 30 AND activity_date <= '2019-07-27'
```



#### ==可以 group by 多个字段，且可以用 Select 查出的东西==

```mysql
GROUP BY Subject, Semester

select day(date) hhh from xx group by hhh
```



#### year(time_stamp) = 2020

```mysql
SELECT user_id, max(time_stamp) last_stamp
FROM Logins
WHERE year(time_stamp) = 2020
GROUP BY user_id

# me
where time_stamp >= '2020-01-01 00:00:00' and time_stamp <= '2020-12-31 23:59:59'
```



#### [sum() 里面可以 case when then end](https://leetcode.cn/problems/capital-gainloss/comments)

```mysql
select stock_name,
sum(case when operation='buy' then -price
else  price  end ) as 'capital_gain_loss'
from Stocks
group by stock_name
```





#### [SQL中 select count(1) count中的1 到底是什么意思呢?和count(*)的区别](https://www.cnblogs.com/xiaoqiqistudy/p/11210716.html)

如果表没有主键，那么count(1)比count(\*)快。 

如果有主键，那么count(key value，union key value)比count(\*)快。 

如果表只有一个字段，count(*)最快。



1、COUNT(字段) 会统计该字段在表中出现的次数，忽略字段为null 的情况。==即不统计字段为null 的记录。== 

2、COUNT(*) 则不同，它执行时返回检索到的行数的计数，不管这些行是否包含null值，

3、COUNT(1)跟COUNT(*)类似，不将任何列是否null列入统计标准，仅用1代表代码行，所以在统计结果的时候，不会忽略列值为NULL的行。



#### between '2019-01-01' and '2019-03-31'



#### 发现函数的下标都是从 1 开始的



#### 字符串匹配指定字串

```mysql
# 180cm,75kg,27,male
select substring_index(profile,',',-1) #SUBSTRING_INDEX(str  ,substr  ,n)：返回字符substr在str中第n次出现位置之前的字符串;
select regexp_substr(profile,"male|female")
```





# 面试没答上来

> 1）Union 和 Union All  --> 具体看上面





> 2）数据库很多请求打过来自己有什么优化方法，除了分表

https://blog.csdn.net/qq_21993785/article/details/81017671  

==慢查询==：查询时间超过了我们设定的时间的语句

1. ***\*SQL查询语句优化\****
2. ***\*主从复制，读写分离，负载均衡\****
3. ***\*数据库分表、分区、分库\****





> 3）分表、分区、分库
>
> 基于上面的补充，浅问了ChartGPT

### 1.概念

MySQL分库分表是一种数据库技术，目的是把大量的数据分散到多个数据库或数据表中，以提高数据库的可扩展性和稳定性。这样做的好处是：

1. 减少单个数据库的负载：分散数据能够缓解单个数据库的压力，防止数据库因为负载过大而瘫痪。
2. 提高可扩展性：通过分散数据到多个数据库或数据表中，可以方便地添加更多的数据库和数据表，以适应增长的数据量。
3. 提高数据的安全性：数据分散到多个数据库或数据表中，每个数据库或数据表的数据量都比较小，如果有一个数据库出现问题，不会影响其他的数据库。

分库分表的方法有很多种，如基于哈希，范围，时间等，具体选择哪种方法取决于实际需求。通常使用中间件（例如Sharding-JDBC）来实现分库分表，使得实现和维护更加简便。

### 2.实现

分库分表的方法有很多种，其中常用的有：

1. 基于哈希：将数据按照一定的算法（例如哈希算法）划分到多个数据库或多个数据表中，这种方法适用于数据分布均匀的情况。
2. 基于范围：将数据按照给定的范围分配到多个数据库或多个数据表中，这种方法适用于数据量较大且有规律的情况。
3. 基于位置：将数据按照数据所在的位置（例如国家、省份）分配到多个数据库或多个数据表中，这种方法适用于对数据位置敏感的情况。
4. 基于时间：将数据按照数据的创建时间或更新时间分配到多个数据库或多个数据表中，这种方法适用于对时间敏感的情况。

不同的应用场景对应不同的分库分表方法，在选择分库分表方法时，需要根据实际情况考虑应用场景、数据量、数据分布情况等因素。
