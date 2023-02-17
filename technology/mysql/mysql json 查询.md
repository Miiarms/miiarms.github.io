# MySQL JSON数据查询

介绍下MySQL json 数据类型的查询方式

## 数据准备



---

## json 相关的方法

### json_extract

`JSON_EXTRACT`  用于根据键提取值，功能有点类似 java 中的 hashMap.get(key)

---

### json_unquote

`JSON_UNQUOTE` 用于去除最外侧的双引号。

---

### 类 *Lambda* 表达式

MySQL 提供了 `->` 和 `->>` 表达式，其中:

- `->`  等价于  `JSON_EXTRACT`，能得到提取结果但是不去除外面的双引号
- `->>` 等价于 `JSON_UNQUOTE(JSON_EXTRACT)`，能得到提取结果且会去除外面的符号

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



