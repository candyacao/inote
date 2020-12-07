## 分页
* ### Mysql分页
  * #### 1. 在MySQL中，分页查询一般都是使用limit子句实现，limit子句声明如下：
```sql
    SELECT * FROM table LIMIT [offset,] rows | rows OFFSET offset
```

LIMIT子句可以被用于指定 SELECT 语句返回的记录数。需注意以下几点：

    * 第一个参数指定第一个返回记录行的偏移量
    * 第二个参数指定返回记录行的最大数目
    * 如果只给定一个参数：它表示返回最大的记录行数目
    * 第二个参数为 -1 表示检索从某一个偏移量到记录集的结束所有的记录行
    * 初始记录行的偏移量是0(而不是 1)
```sql
    select * from orders_history where type=8 limit 1000,10;
    --该条语句将会从表 orders_history 中查询第1000条数据之后的10条数据，也就是第1001条到第1010条数据。

    --数据表中的记录默认使用主键（一般为id）排序，上面的结果相当于：

    select * from orders_history where type=8 order by id limit 10000,10;
    --在查询记录量低于100时，查询时间基本没有差距，随着查询记录量越来越大，所花费的时间也会越来越多。
    --这种分页查询方式会从数据库第一条记录开始扫描，所以越往后，查询速度越慢，而且查询的数据越多，也会拖慢总查询速度
```
  * #### 2. 使用子查询优化
```sql
    --这种方式先定位偏移位置的 id，然后往后查询，这种方式适用于 id 递增的情况。

    select * from orders_history where type=8 limit 100000,1;

    select id from orders_history where type=8 limit 100000,1;

    select * from orders_history where type=8 and id>=(select id from orders_history where type=8 limit 100000,1) limit 100;

    select * from orders_history where type=8 limit 100000,100;
```

针对上面的查询需要注意：

1、比较第1条语句和第2条语句：使用 select id 代替 select * 速度增加了3倍

2、比较第2条语句和第3条语句：速度相差几十毫秒

3、比较第3条语句和第4条语句：得益于 select id 速度增加，第3条语句查询速度增加了3倍

这种方式相较于原始一般的查询方法，将会增快数倍。
  * #### 3. 使用 id 限定优化

    这种方式假设数据表的id是连续递增的，则我们根据查询的页数和查询的记录数可以算出查询的id的范围，可以使用 id between and 来查询：
```sql
    select * from orders_history where type=2 and id between 1000000 and 1000100 limit 100;
```

这种查询方式能够极大地优化查询速度，基本能够在几十毫秒之内完成。限制是只能使用于明确知道id的情况，不过一般建立表的时候，都会添加基本的id字段，这为分页查询带来很多便利。

还可以有另外一种写法：
```sql
select * from orders_history where id >= 1000001 limit 100;
```
当然还可以使用 in 的方式来进行查询，这种方式经常用在多表关联的时候进行查询，使用其他表查询的id集合，来进行查询：
```sql
select * from orders_history where id in
(select order_id from trade_2 where goods = 'pen')
limit 100;
```
这种 in 查询的方式要注意：某些 mysql 版本不支持在 in 子句中使用 limit。

关于数据表的id说明：

一般情况下，在数据库中建立表的时候，每一张表强制添加 id 递增字段，这样更方便我们查询数据。

如果数据量很大，比如像订单这类，一般会推荐进行分库分表。这个时候 id 就不建议作为唯一标识了，而应该使用分布式的高并发唯一 id 生成器来生成，并在数据表中使用另外的字段来存储这个唯一标识。

首先使用范围查询定位 id （或者索引），然后再使用索引进行定位数据，即先 select id，然后在 select *；这样查询的速度将会提升好几倍。
* ### Oracle分页
  * #### 1. 采用伪列rownum
```SQl
    --查询前10条记录
    select * from t_user t where ROWNUM <10;
    --按照学生ID排名，抓取前三条记录
    SELECT * FROM(SELECT id,realname FROM T_USER ORDER BY id asc ) WHERE ROWNUM <=3
    --分页SQL写法，从第10条记录开始，提取10条记录。
    SELECT * FROM (SELECT ROWNUM rn,id,realname FROM (SELECT id,realname FROM T_USER)WHERE ROWNUM<=20) t2 WHERE T2.rn >=10;
    --按照学生ID排名，从第10条记录开始，提取10条记录。
    SELECT * FROM (SELECT ROWNUM rn,id,realname FROM (SELECT id,realname FROM T_USER ORDER BY id asc)WHERE ROWNUM<=20) t2 WHERE T2.rn >=10;

```
**【注】**

    1. where rownum>1 不能抓取到记录。
   
    2. where rownum between 2 and 10 也不能抓取到记录。
  * #### 2. 运用分析函数
    用分析函数row_number()over(ORDER BY 字段)
```Sql
    --按照学生ID排名，抓取前三条记录
    SELECT * FROM(SELECT id,realname,row_number()over(ORDER BY id asc) rn FROM T_USER)WHERE rn <=3
    --按照学生ID排名，从第10条记录开始，提取10条记录。
    SELECT * FROM(SELECT id,realname,row_number()over(ORDER BY id asc) rn FROM T_USER)WHERE rn BETWEEN 10 AND 20
```
  * #### 3. 运用minus方法
```sql
    --从第10条记录开始，提取10条记录。
    SELECT * FROM T_USER WHERE ROWNUM<20 MINUS SELECT * FROM T_USER WHERE ROWNUM<10;
    --按ID排序后，从第10条记录开始，提取10条记录。
    (SELECT * FROM (SELECT * FROM T_USER ORDER BY id asc) WHERE ROWNUM<20)  MINUS( SELECT * FROM (SELECT * FROM T_USER ORDER BY id asc) WHERE ROWNUM<10);
```