# 统计查询SQL

```sql
 SELECT
	region_. "name" AS area_name,
	region1_. "name" AS station_name,
	hall_. "name" AS hall_name,
	hall_.hall_type AS hall_type,
	hall_.use_area AS use_area,
	hall_.seat_count AS seat_count,
	COALESCE(SUM(order_info_.amount), 0.00) AS amount,
	COALESCE(SUM(order_info_.actual_amount), 0.00) AS actual_amount,
	sum(
		CASE WHEN sub_ref.commodity_type IN(1, 5, 6) THEN
			sub_ref.amount
		ELSE
			0.00
		END) AS card_amount,
	sum(
		CASE WHEN sub_ref.commodity_type = 2 THEN
			sub_ref.amount
		ELSE
			0.00
		END) AS appreciate_amount,
	sum(
		CASE WHEN sub_ref.commodity_type = 3 THEN
			sub_ref.amount
		ELSE
			0.00
		END) AS travel_amount,
	sum(
		CASE WHEN sub_ref.commodity_type = 4 THEN
			sub_ref.amount
		ELSE
			0.00
		END) AS agent_amount
FROM
	hall hall_
	LEFT OUTER JOIN region region_ ON SUBSTRING(hall_.region_long_code, 1, 8) = region_.long_code
	LEFT OUTER JOIN region region1_ ON hall_.region_long_code = region1_.long_code
	LEFT OUTER JOIN order_info order_info_ ON order_info_.sale_hall_code = hall_.code
	LEFT OUTER JOIN sub_order_commodity_ref sub_ref ON sub_ref.order_id = order_info_.id
WHERE
	1 = 1
GROUP BY
	region_. "name",
	region1_. "name",
	hall_. "name",
	hall_.hall_type,
	hall_.use_area,
	hall_.use_area,
	hall_.seat_count
LIMIT 10 OFFSET 0
```

统计站厅销售金额

COALESCE列为空赋值0.00