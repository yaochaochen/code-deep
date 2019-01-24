今天一位开发的同事给我一个SQL, 问我为什么只改了一个条件, 查询速度居然从毫秒就慢到几十秒了,
SELECT *                                                                                  
  FROM tbl  
  where create_time>='2014-02-08' and create_time<'2014-02-11'  
  and x=3  
  and id != '123'  
  and id != '321'  
  and y > 0 order by create_time limit 1 offset 0; 
  执行结果100毫秒
  改成如下 :
  SELECT *                                                                                  
  FROM tbl  
  where create_time>='2014-02-08' and create_time<'2014-02-11'  
  and x=3  
  and id != '123'  
  and id != '321'  
  and y > 0 order by create_time limit 1 offset 10;  
  耗时居然高达几十秒
  执行了SQL计划是完全一样的，为什么导致执行时间差异巨大的问题
  所以第二个SQL到底扫描了多少行？
  我查询第二个create_time值多少 很诧异
  select create_time from tbl   
  where create_time>='2014-02-08' and create_time<'2014-02-11'  
  and x=3  
  and id != '123'  
  and id != '321'  
  and y > 0 order by create_time limit 1 offset 10;  
  执行结果：'2014-02-08 18:38:35.79'  
  第二个SQL扫描行数如下
  select count(*) from tbl where create_time<='2014-02-08 18:38:35.79' and create_time>='2014-02-08';  
  count    
---------  
 1448081  
(1 row)  
也就是说crate_time这个字段分散比较零散，并且数据量比较庞大，所以offset10后走crate_time索引自然很慢了

下面优化
在不新增任何索引的前提下, 还是走create_time这个索引, 减少重复扫描的数据.

需要得到每次取到的最大的create_time值, 以及可以标示这条记录的唯一ID.

下次取的时候, 不要使用offset 下一页, 而是加上这两个条件.
select create_time from tbl   
  where create_time>='2014-02-08' and create_time<'2014-02-11'  
  and x=3  
  and id != '123'  
  and id != '321'  
  and pk not in (?)  -- 这个ID是上次取到的create_time最大的值的所有记录的pk值.  
  and y > 0   
  and create_time >= '2014-02-08 18:38:35.79'  -- 这个时间是上次取到的数据的最大的时间值.  
  order by create_time limit ? offset 0;  
  通过这种方法，可以减少limit offset 的离散扫描
  

