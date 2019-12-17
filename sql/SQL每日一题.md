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

