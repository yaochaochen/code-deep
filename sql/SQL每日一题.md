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

