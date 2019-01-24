昨天在一个业务库中发现一个比较耗时的SQL, 如下 :
select t.APP_ID, t.APP_VER, t.CN_NAME, t.PACKAGE, t.APK_SIZE, t.APP_SHOW_VER, t.DESCRIPTION,t.CONTENT_PROVIDER,at.APP_TAG,h.SCORE       
from   
(select APP_ID,max(APP_VER) APP_VER from test1 group by APP_ID) s  
join test1 t  
on s.APP_ID=t.APP_ID and s.APP_VER=t.APP_VER and t.DELETED=0    
left outer join test2 at   
on t.APP_ID=at.APP_ID       
left outer join   
test3 h   
on t.APP_ID=h.APP_ID       
limit 24 offset 0;  
注意到这里面有一个子查询select APP_ID,max(APP_VER) APP_VER from test1 group by APP_ID, 用来取出app_id上面的max(app_ver).

也就是要检索的是最大版本的app_id. 每个表上的app_id上都有索引.

优化1
消除子查询, select APP_ID,max(APP_VER) APP_VER from test1 group by APP_ID .

这是个读多写少的表, 所以可以这么来优化.

通过增加一个ismax字段, 标记该app_id的app_ver是否是max版本. 因此需要建立一个触发器来完成这个字段的更新, 确保最新的状态.

另外需要一个约束, 确保不会出现ismax重复为true的情况. (由于并发的情况下这个比较难保证, 新增的数据可能都会认为自己是max的版本,所以还有优化2).

优化2
通过优化1, 查询性能基本上有1倍左右的提升.

但是从总时长来看, 越到后面, 每一次翻页都需要消耗几百毫秒. 每页显示的数量越少, 全部翻完所消耗的时间就越长.

我们来用一个函数测试一下使用大小不一样的分页, 看看时间分别是多少.

create or replace function f_test1(i_limit int) returns int as $$  
declare  
v_work int;  
v_count int;  
v_offset int;  
i int;  
begin  
v_work := 0;  
v_count := 0;  
v_offset := 0;  
i := 1;  
raise notice 'start time:%',clock_timestamp();  
loop  
  select count(*) into v_work from   
    (select 1 from   
      test1 t  
      left outer join test2 at   
      on (t.APP_ID=at.APP_ID and t.DELETED=0 and t.ismax is true)  
      left outer join   
      test3 h   
      on (t.APP_ID=h.APP_ID)  
      limit i_limit offset v_offset  
    )t;  
  if v_work=0 then  
    exit;  
  end if;  
  v_offset := i * i_limit;  
  v_count := v_count + v_work;  
  i := i+1;  
end loop;  
raise notice 'end time:%',clock_timestamp();  
return v_count;  
end;  
$$ language plpgsql;  
  
-- 测试每页10000条  
digoal=> select * from f_test1(10000);  
NOTICE:  start time:2012-06-21 10:44:34.148575+08  
NOTICE:  end time:2012-06-21 10:44:36.095619+08  
 f_test1   
---------  
  110646  
(1 row)  
耗时2秒.  
  
-- 测试每页100条  
digoal=> select * from f_test1(100);  
NOTICE:  start time:2012-06-21 10:44:45.448933+08  
NOTICE:  end time:2012-06-21 10:46:53.070959+08  
 f_test1   
---------  
  110646  
(1 row)  
耗时128秒. 
所以第二种优化手段是, 提高每页获取的数量, 增加应用层缓存, 也就是说每次取的页数多一点, 不用每一页都来数据库取. 当然如果应用能够一次把数据全取过去就最好了.

最后一点, 本例的分页SQL都没有ORDER BY, 算是个业务层的BUG, 这种分页是不可取的, 因为无法保证返回的顺序.
