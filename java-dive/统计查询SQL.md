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

## 统计站厅销售金额

COALESCE列为空赋值0.00



```sql
 SELECT
	region_. "name" AS area_name,
	region1_. "name" AS station_name,
	hall_. "name" AS hall_name,
	hall_.hall_type AS hall_type,
	hall_.use_area AS use_area,
	hall_.seat_count AS seat_count,
	COUNT(order_info_) AS reception_count,
	SUM(
		CASE WHEN order_info_.svc_source_name = '扫码接待' THEN
			1
		ELSE
			0
		END) AS reception_scan_count,
	SUM(
		CASE WHEN order_info_.svc_source_name = '商务票' THEN
			1
		ELSE
			0
		END) AS reception_business_count,
	SUM(
		CASE WHEN order_info_.svc_source_name = '会员' THEN
			1
		ELSE
			0
		END) AS reception_member_count,
	SUM(
		CASE WHEN order_info_.svc_source_name = '散客' THEN
			1
		ELSE
			0
		END) AS reception_individual_count,
	SUM(
		CASE WHEN order_info_.svc_source_name NOT IN('散客', '会员', '商务票', '扫码接待') THEN
			1
		ELSE
			0
		END) AS reception_company_count
FROM
	hall hall_
	LEFT OUTER JOIN region region_ ON SUBSTRING(hall_.region_long_code, 1, 8) = region_.long_code
	LEFT OUTER JOIN region region1_ ON hall_.region_long_code = region1_.long_code
	LEFT OUTER JOIN svc_order order_info_ ON order_info_.recept_hall_code = hall_.code
		AND order_info_.svc_status != 5
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
ORDER BY
	reception_count DESC
```
## 按照站厅和时间维度统计
```sql SELECT
	sum(DD.new_fans) AS new_fans,
	sum(dd.old_fans) AS old_fans,
	dd.hall_name,
	dd.create_date AS create_time
FROM (
	SELECT
		count(1) AS new_fans,
		'0' AS old_fans,
		hall_name,
		to_char(create_time, 'yyyy-MM-dd') AS create_date
	FROM
		hall_attention
	WHERE
		"event" = 0
	GROUP BY
		hall_name,
		create_date
	UNION
	SELECT
		'0' AS new_fans,
		count(1) AS old_fans,
		hall_name,
		to_char(create_time, 'yyyy-MM-dd') AS create_date
	FROM
		hall_attention
	WHERE
		event = 1
	GROUP BY
		hall_name,
		create_date) AS dd
GROUP BY
	dd.hall_name,
	dd.create_date
ORDER BY
	dd.create_date DESC ```

