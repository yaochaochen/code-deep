业务场景
查询前10的优惠劵码的使用的所属活动

测试表和测试数据，生成10000个分组，1000万条记录。

postgres=# create table tbl(c1 int, c2 int, c3 int);
CREATE TABLE
postgres=# create index idx1 on tbl(c1,c2);
CREATE INDEX
postgres=# insert into tbl select mod(trunc(random()*10000)::int, 10000), trunc(random()*10000000) from generate_series(1,10000000);
INSERT 0 10000000

使用窗口查询的执行计划

postgres=# explain select * from (select row_number() over(partition by c1 order by c2) as rn,* from tbl) t where t.rn<=10;
                                       QUERY PLAN                                       
----------------------------------------------------------------------------------------
 Subquery Scan on t  (cost=0.43..770563.03 rows=3333326 width=20)
   Filter: (t.rn <= 10)
   ->  WindowAgg  (cost=0.43..645563.31 rows=9999977 width=12)
         ->  Index Scan using idx1 on tbl  (cost=0.43..470563.72 rows=9999977 width=12)
(4 rows)

使用窗口查询的效率，20.1秒

如何优化？ 
使用递归查询，优化count distinct的方法。 
plain (analyze,verbose,timing,costs,buffers) select * from f();
                                                     QUERY PLAN                                                      
---------------------------------------------------------------------------------------------------------------------
 Function Scan on public.f  (cost=0.25..10.25 rows=1000 width=12) (actual time=419.218..444.810 rows=100000 loops=1)
   Output: c1, c2, c3
   Function Call: f()
   Buffers: shared hit=170407, temp read=221 written=220
 Planning time: 0.037 ms
 Execution time: 464.257 ms
(6 rows)
优化后，只需要464毫秒返回10000个分组的TOP 10。
传统的方法使用窗口查询，输出多个每个分组的TOP 10，需要扫描所有的记录。效率较低。
由于分组不是非常多，只有10000个，所以可以选择使用递归的方法，用上索引取TOP 10，速度非常快。
目前PostgreSQL的递归语法不支持递归的启动表写在subquery里面，也不支持启动表在递归查询中使用order by，所以不能直接使用递归得出结果，目前需要套一层函数。
