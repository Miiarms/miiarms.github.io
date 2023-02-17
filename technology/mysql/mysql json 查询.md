# MySQL JSON数据查询

介绍下MySQL json 数据类型的查询方式

## 数据准备

创建一个配置变更表，并初始化三条数据做演示。

```sql
create table config_snapshot
(
    id            int auto_increment comment '自增主键'
        primary key,
    snapshot_json json        null comment '配置快照',
    city_ids      json        null comment '城市编码数组',
    operator      varchar(20) null comment '操作人',
    create_time   datetime    null comment '创建时间'
)
    comment '配置变更,测试json查询';

-- 初始化三条数据供演示
INSERT INTO gac_member.config_snapshot (id, snapshot_json, city_ids, operator, create_time) VALUES (1, '{"id": 1001, "city_ids": ["440100", "440300", "440400"], "operator": "操作员1", "create_time": "2023-02-16 10:00:23", "snapshot_json": [{"cityCode": "430100", "priceRate": {"ceil": 98, "floor": 0.5, "diffValue": 0.1}, "supplierList": [{"supplierId": 13, "supplierName": "供应商13", "operationTime": 1675319819000}, {"supplierId": 9, "supplierName": "供应商9", "operationTime": 1675319819000}, {"supplierId": 12, "supplierName": "供应商12", "operationTime": 1675319819000}]}, {"cityCode": "440100", "priceRate": {"ceil": 98, "floor": 0.5, "diffValue": 10}, "supplierList": [{"supplierId": 13, "supplierName": "供应商13", "operationTime": 1673403841000}]}]}', '["440100", "440300", "440400"]', '操作员1', '2023-02-16 10:00:23');
INSERT INTO gac_member.config_snapshot (id, snapshot_json, city_ids, operator, create_time) VALUES (2, '{"id": 1001, "city_ids": ["440100", "430300", "440110"], "operator": "操作员1", "create_time": "2023-02-16 10:00:23", "snapshot_json": [{"cityCode": "430300", "priceRate": {"ceil": 98, "floor": 0.5, "diffValue": 0.1}, "supplierList": [{"supplierId": 2, "supplierName": "供应商2", "operationTime": 1675319819000}, {"supplierId": 91, "supplierName": "供应商91", "operationTime": 1675319819000}, {"supplierId": 125, "supplierName": "供应商125", "operationTime": 1675319819000}]}, {"cityCode": "440110", "priceRate": {"ceil": 98, "floor": 0.5, "diffValue": 10}, "supplierList": [{"supplierId": 13, "supplierName": "供应商13", "operationTime": 1673403841000}]}]}', '["440100", "440310", "440410"]', '操作员2', '2023-02-16 11:20:16');
INSERT INTO gac_member.config_snapshot (id, snapshot_json, city_ids, operator, create_time) VALUES (3, '{"id": 1001, "city_ids": ["440400", "440700", "440200"], "operator": "操作员1", "create_time": "2023-02-16 10:00:23", "snapshot_json": [{"cityCode": "430100", "priceRate": {"ceil": 98, "floor": 0.5, "diffValue": 0.1}, "supplierList": [{"supplierId": 17, "supplierName": "供应商17", "operationTime": 1675319819000}, {"supplierId": 11, "supplierName": "供应商11", "operationTime": 1675319819000}]}]}', '["440400", "440800", "441400"]', '操作员3', '2023-02-16 13:10:43');

```

`snapshot_json`  字段结构示例：

```json
{
	"id": 1001,
	"operator": "操作员1",
	"snapshot_json": [{
		"cityCode": "430100",
		"supplierList": [{
			"supplierId": 13,
			"supplierName": "供应商13",
			"operationTime": 1675319819000
		}, {
			"supplierId": 9,
			"supplierName": "供应商9",
			"operationTime": 1675319819000
		}, {
			"supplierId": 12,
			"supplierName": "供应商12",
			"operationTime": 1675319819000
		}],
		"priceRate": {
			"ceil": 98,
			"floor": 0.5,
			"diffValue": 0.1
		}
	}, {
		"cityCode": "440100",
		"supplierList": [{
			"supplierId": 13,
			"supplierName": "供应商13",
			"operationTime": 1673403841000
		}],
		"priceRate": {
			"ceil": 98,
			"floor": 0.5,
			"diffValue": 10
		}
	}],
	"city_ids": ["440100", "440300", "440400"],
	"create_time": "2023-02-16 10:00:23"
}
```



---

## json 相关的方法

### json_extract

`JSON_EXTRACT`  用于根据键提取值，功能有点类似 java 中的 hashMap.get(key)

**案例演示：**

- 获取json中的id字段

  ```sql
  select json_extract(snapshot_json, '$.id') from config_snapshot;
  ```

  结果如下：

  <img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302171758872.png" alt="image-20230217175800659" style="zoom:33%;" />

- 获取json 里面的city_code字段

  获取下面这个`cityCode`字段的所有值

  <img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302171801873.png" alt="image-20230217180116815" style="zoom:33%;" />

  ```sql
  select json_extract(snapshot_json, '$.snapshot_json[*].cityCode') from config_snapshot;
  ```

  运行结果：

  <img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302171807149.png" alt="image-20230217180717095" style="zoom:33%;" />

  > <font color="red">上面的SQL 中， `$`  符号表示当前字段本身;  `*` 符号表示数组数据时，所有数组下标，亦可以用0,1,2等数字符号表示数组的下标</font>

---

### json_unquote

`JSON_UNQUOTE` 用于去除最外侧的双引号。

**案例演示：**

```sql
-- 因为查询operator字段是字符串，结果会有 JSON 的双引号
select json_extract(snapshot_json, '$.operator') from config_snapshot;
-- json_unquote会去掉查询结果的双引号
select json_unquote(json_extract(snapshot_json, '$.operator')) from config_snapshot;
```

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302171823340.png" alt="image-20230217182350264" style="zoom:33%;" />

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302171824739.png" alt="image-20230217182426666" style="zoom:33%;" />

---

### 类 *Lambda* 表达式

MySQL 提供了 `->` 和 `->>` 表达式，其中:

- `->`  等价于  `JSON_EXTRACT`，能得到提取结果但是不去除外面的双引号
- `->>` 等价于 `JSON_UNQUOTE(JSON_EXTRACT)`，能得到提取结果且会去除外面的符号

**案例演示：**

   ```sql
   -- 因为查询operator字段是字符串，结果会有 JSON 的双引号
   select snapshot_json ->'$.operator' from config_snapshot;
   -- json_unquote会去掉查询结果的双引号
   select snapshot_json ->>'$.operator' from config_snapshot;
   ```

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302171831260.png" alt="image-20230217183133186" style="zoom:33%;" />

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302171832791.png" alt="image-20230217183201722" style="zoom:33%;" />

---

### json_contains

有两种功能：

- 对JSON数组检查一个元素或者多个元素是否存在；
- 对于JSON对象检查指定KEY是否有某个值

---

### json_overlap（MySQL8.0+）

比较两个JSON数组是否至少有一个元素一致，如果是返回1，否则返回0，如果是JSON对象，判断是否是有一对key value一致。

----

### member of（MySQL8.0+）

匹配某个元素是否存在，返回1表示元素存在，返回0表示元素不存在

---

## json 复杂查询



```sql
select 
       json_extract(config_update_before, '$.urbanConfig') as before_update ,
       json_extract(config_update_after, '$.urbanConfig') as after_update
from open_config_snapshot 
where 
	json_contains(json_extract(config_update_before, '$.urbanConfig[*].supplierList[*].supplierId'), '[10]' , '$')
   -- 哪个城市
   and json_contains(json_extract(config_update_before, '$.urbanConfig[*].cityCode'), '"440600"' , '$')
 
```



