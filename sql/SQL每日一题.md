# SQL每日一题

## 2019-12-12

编写一个SQL查询，查询所有与至少连续出现两次的数字z

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

