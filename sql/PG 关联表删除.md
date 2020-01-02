# PG 关联表删除



```sql
DELETE FROM order_info o USING sub_order_commodity_ref f
WHERE f.order_no = o.order_no and f.sub_order_detail_id is null		
```

s