------

# SQL每日一题

## 2019-12-12

编写一个SQL查询，查询所有与至少连续出现两次的数字

| Id   | Num  |
| ---- | ---- |
| 1    | 1    |
| 2    | 1    |
| 3    | 1    |
| 4    | 2    |
| 5    | 1    |
| 6    | 2    |
| 7    | 2    |
| 8    | 3    |
| 9    | 4    |

例如，给定上面的logs表，1和2是连续出现至少两次的数字

| Num  | Times |
| ---- | ----- |
| 1    | 3     |
| 2    | 2     |

```sql
select num , count(1) as count from ( select id , num , row_number() over(partition by num order by id) as rank_num
from tmp.table_num ) t1 group by num,id - rank_num having count(1) > 1 ;
```

```sql
 select t1.num as Num,sum(case when t1.num = t2.num then 1 else 0 end) + 1 as Times
from Logs t1 left join Logs t2 on t1.id + 1 = t2.id group by t1.num	
```

------

## 2019-12-13

**有如下一张记录表，如何查询出每隔15分钟的记录数**

| ID   | Times            |
| ---- | ---------------- |
| 1    | 2019/12/13 11:01 |
| 2    | 2019/12/13 11:03 |
| 3    | 2019/12/13 11:05 |
| 4    | 2019/12/13 11:09 |
| 5    | 2019/12/13 11:17 |
| 6    | 2019/12/13 11:19 |
| 7    | 2019/12/13 11:29 |
| 8    | 2019/12/13 11:37 |

**预计结果**

| 时间段           | 行数 |
| ---------------- | ---- |
| 2019/12/13 11:00 | 4    |
| 2019/12/13 11:15 | 3    |
| 2019/12/13 11:30 | 1    |

```sql
select hours||':'||times as times, count(1) as rows from ( select split_part(times,':',1) as hours , case when split_part(times,':',2) < 15 then '00' when split_part(times,':',2) < 30 then '15' when split_part(times,':',2) < 45 then '30' else '45' end as times from tmp.table_time ) t1 group by 1 ;
```

------

## 2019-12-16

几个朋友来到电影院的售票处，准备预约连续空余座位。

你能利用表 cinema ，帮他们写一个查询语句，获取所有空余座位，并将它们按照 seat_id 排序后返回吗？

| seat_id | free |
| ------- | ---- |
| 1       | 1    |
| 2       | 0    |
| 3       | 1    |
| 4       | 1    |
| 5       | 1    |

对于如上样例，你的查询语句应该返回如下结果。

| seat_id |
| ------- |
| 3       |
| 4       |
| 5       |

注意：
seat_id 字段是一个自增的整数，free 字段是布尔类型（'1' 表示空余， '0' 表示已被占据）。
连续空余座位的定义是大于等于 2 个连续空余的座位。

```sql
SELECT 
	DISTINCT c1.seat_id
FROM 
	cinema c1
LEFT JOIN 
	cinema c2
ON 
	abs(c1.seat_id - c2.seat_id) = 1
WHERE 
	c1.free = 1 AND c2.free = 1
ORDER BY c1.seat_id;
```

------

## 2019-12-17

有如下三列和几组数据

| A    | B    | C    |
| ---- | ---- | ---- |
| aa   | 1    | X    |
| aaa  | 2    | Y    |
| bbb  | 3    | X    |
| bbb  | 4    | X    |
| ccc  | 5    | Y    |
| ccc  | 6    | Y    |
|      |      |      |

想得到如下结果

| A    | B    | C    |
| ---- | ---- | ---- |
| aaa  | 3    | 1    |
| bbb  | 7    | X    |
| ccc  | 11   | Y    |

```sql
 select a,b,c from (select a , sum(b) over(partition by a) as b , case when lag(c,1) over(partition by a order by b) = c then c else '1' end as c , row_number() over(partition by a order by b desc) as rank_num from tmp.table1 ) t1 where t1.rank_num = 1 ;
```

------

## 2019-12-18

有如下两张表:

Project 表

| project_id | employee_id |
| ---------- | ----------- |
| 1          | 1           |
| 1          | 2           |
| 1          | 3           |
| 2          | 1           |
| 2          | 4           |

Employee表:

| employee_id | Name   | Experience_years |
| ----------- | ------ | ---------------- |
| 1           | Khaled | 3                |
| 2           | Ali    | 2                |
| 3           | John   | 3                |
| 4           | Doe    | 2                |

查询出每个项目中经验最丰富(experience_years最大)的员工

| project_id | employee_id |
| ---------- | ----------- |
| 1          | 1           |
| 1          | 3           |
| 2          | 1           |

说明：员工1和3是project_id为1中exprerience_years最丰富的,
而project_id为2的项目，员工id为1的是exprerience_years最丰富

```sql
select project_id , employee_id from (select t1.project_id , t1.employee_id , t2.experience_years , rank() over(partition by t1.project_id order by t2.experience_years desc) as rank_num from tmp.table_project t1 join tmp.table_employee t2 on t1.employee_id = t2.employee_id ) a where a.rank_num = 1
```

## 2019-12-20

如下一张表

ActorDirector

| actor_id | director_id | timestamp |
| -------- | ----------- | --------- |
| 1        | 1           | 0         |
| 1        | 1           | 1         |
| 1        | 1           | 2         |
| 1        | 2           | 3         |
| 1        | 2           | 4         |
| 2        | 1           | 5         |
| 2        | 1           | 6         |

写一条SQL查询语句获取合作过至少三次的演员和导演的ID对 actor_id 和 director_id

结果:

| actor_id | director_id |
| -------- | ----------- |
| 1        | 1           |

```sql
 select action_id , director_id from tmp.actordirector group by 1,2 having count(1)>=3 	
```

## 2019-12-23

你能写一个 SQL 查询语句，找到只出现过一次的数字中，最大的一个数字吗？下面是测试数据

| Num  |
| ---- |
| 8    |
| 8    |
| 3    |
| 3    |
| 1    |
| 4    |
| 5    |
| 6    |
| 7    |

对于上面给出的样例数据，你的查询语句应该返回如下结果：

| Num  |
| ---- |
| 6    |

```sql
select max(num) as num from (select num from table1 group by num having count(1) = 1 ) t1 ;
```

## 2019-12-27

表 orders 定义如下，order_id 订单编号， customer_id  客户编码 order_date 下单时间

有如下几条记录

| order_id | customer_id | order_date |
| -------- | ----------- | ---------- |
| 1        | 1           | 20190624   |
| 2        | 2           | 20190423   |
| 3        | 3           | 20190321   |
| 4        | 3           | 20190429   |
| 5        | 4           | 20190812   |
| 6        | 4           | 20190914   |

在 orders 中找到订单数最多的客户对应的 customer_id 

预计输出

| customer_id |
| ----------- |
| 3           |
| 4           |

```sql
select customer_id from orders group by customer_id having count(*) = (
select count(*) from orders o group by o.customer_id order by count(*) desc
limit 1)
```

## 2019-12-31

有如下几张表



Student

| S#   | Sname | Sage       | Ssex |
| ---- | ----- | ---------- | ---- |
| 01   | 赵雷  | 1990-1-1   | 男   |
| 02   | 钱电  | 1990-12-21 | 男   |
| 03   | 李云  | 1990-8-6   | 男   |
| 04   | 孙风  | 1990-5-20  | 男   |
| 05   | 周梅  | 1991-12-1  | 女   |
| 06   | 吴兰  | 1992-3-1   | 女   |
| 07   | 郑竹  | 1989-7-1   | 女   |
| 08   | 王菊  | 1990-1-20  | 女   |

Course

| C#   | Cname | T#   |
| ---- | ----- | ---- |
| 01   | 语文  | 02   |
| 02   | 数学  | 01   |
| 03   | 英语  | 03   |

SC

| S#   | C#   | score |
| ---- | ---- | ----- |
| 01   | 01   | 80    |
| 01   | 02   | 90    |
| 01   | 03   | 99    |
| 02   | 01   | 70    |
| 02   | 02   | 60    |
| 03   | 01   | 80    |
| 03   | 02   | 80    |
| 03   | 03   | 80    |
| 04   | 01   | 50    |
| 04   | 02   | 30    |
| 04   | 03   | 20    |
| 05   | 01   | 87    |
| 05   | 02   | 87    |
| 06   | 01   | 31    |
| 06   | 03   | 34    |
| 07   | 02   | 89    |
| 07   | 03   | 89    |
|      |      |       |
|      |      |       |

查询"01 "课程比" 02 "课程成绩高的学生的信息及课程分数？

```sql
SELECT * FROM students_score t1 WHERE NOT EXISTS (SELECT 1 FROM students_score t WHERE t1.NAME = t.NAME AND t.SCORE < 80 GROUP BY NAME )	
```

## 2020-01-02

给定一个 tree 表，id 是树节点编号， p_id 是它父节点的 id 

| ID   | PID  |
| ---- | ---- |
| 1    | NULL |
| 2    | 1    |
| 3    | 1    |
| 4    | 2    |
| 5    | 2    |

树中每个节点属于以下三种类型之一：叶子：如果这个节点没有任何孩子节点。根：如果这个节点是整棵树的根，即没有父节点。内部节点：如果这个节点既不是叶子节点也不是根节点 

写一个查询语句，输出所有节点的编号和节点的类型，并将结果按照节点编号排序。上面样例的结果为：

| ID   | Type  |
| ---- | ----- |
| 1    | Root  |
| 2    | Inner |
| 3    | Leaf  |
| 4    | Leaf  |
| 5    | Leaf  |

解释：
节点 '1' 是根节点，因为它的父节点是 NULL ，同时它有孩子节点 '2' 和 '3' 。节点 '2' 是内部节点，因为它有父节点 '1' ，也有孩子节点 '4' 和 '5' 。节点 '3', '4' 和 '5' 都是叶子节点，因为它们都有父节点同时没有孩子节点。注意 如果树中只有一个节点，你只需要输出它的根属性。

```sql

```

## 2020-01-13

有如下一张表  Order

| shipid | paydate   | payno |
| ------ | --------- | ----- |
| 1001   | 2019/11/2 | 5     |
| 1001   | 2019/11/2 | 3     |
| 1001   | 2019/11/3 | 1     |
| 1001   | 2019/11/3 | 3     |
| 1002   | 2019/11/9 | 1     |
| 1002   | 2019/11/9 | 4     |
| 1002   | 2019/11/8 | 3     |
| 1002   | 2019/11/8 | 2     |
|        |           |       |

查询出每个发货单号(shipid)，最早付款时间(paydate)和最小付款单号(payno)

结果如下

| shipid | paydate   | payno |
| ------ | --------- | ----- |
| 1002   | 2019/11/2 | 3     |
| 1002   | 2019/11/8 | 2     |

```sql
select * from Orders 
where id in (
select SUBSTRING_INDEX(GROUP_CONCAT(id ORDER BY pay_date,pay_no),',',1) from Orders
GROUP BY shop_id
)
```

## 2020-01-14

编写一个SQL查询，报告在首次登录后第二天再次登录的玩家的分数，四舍五入到小数点后两位。换句话说，你需要从首次登录日期开始计算至少连续两天登录的玩家数，把这个数字除以玩家总数。

查询结果格式如下所示：

| play_id | device_id | event_time | game_played |
| ------- | --------- | ---------- | ----------- |
| 1       | 2         | 2016/3/1   | 5           |
| 1       | 2         | 2016/3/2   | 6           |
| 2       | 3         | 2017/6/25  | 1           |
| 3       | 1         | 2016/3/2   | 0           |
| 3       | 4         | 2018/7/3   | 5           |

Result 表

1

| fraction |
| -------- |
| 0.33     |

