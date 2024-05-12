---
title: 常用SQL指南   
date: 2023-06-18 14:36:50   
tags: mysql   
categories: mysql   
excerpt: 日常开发工作中，经常需要和SQL打交道，好的SQL姿势可以事半功倍.
---

### 多行合成一行
```shell
select `daily_plan_id`, GROUP_CONCAT( `pp_id` SEPARATOR ',') as pp_ids, count(1) FROM `table_a` GROUP BY `daily_plan_id`;

select group_concat(id) from table_a where repo in ('xxxx', 'yyyy') and type = 'push' and enabled = 1;
```

### 多列合成一列
```shell
select
  concat_ws('#', name, id, git_id, `git_namespace`)
FROM
  `table_a`
where
  name in (
    'xxxx',
    'yyyy',
    'zzzz'
  );
```

### Case When语句
```shell
# case 1
select * from (select id, created_at, plan_id, ref, commit, CASE status
    WHEN 2 THEN '成功'
    WHEN 3 THEN '失败'
    ELSE '运行中'
  END as '流水线状态', repo, repo_id from table_a where plan_id in (223,453) and created_at BETWEEN "2023-03-23 00:00:00" and "2023-06-10 00:00:00" and (ref like "release%" or ref like "test%") and status in (1, 2, 3)) as cp left join (select `name`, type, pipeline_id, CASE status
    WHEN 0 THEN '等待执行'
    WHEN 1 THEN '初始化中'
    WHEN 2 THEN '执行中'
    WHEN 3 THEN '成功'
    WHEN 4 THEN '失败'
    WHEN 5 THEN '取消'
    ELSE '未知'
  END as '任务状态' from table_b where type = 'build') as cj on cp.id = cj.pipeline_id;
# case 2
select bus, sum(num) from (
    select case
        when belong_id in (12) then 'xxx'
        when belong_id in (44,213) then 'vvv'
        else '未知'
        end as bus, count(repo) as num from table_a where repo in (
        'xxxx/yyyy', 'yyyy/zzzz'
        ) GROUP BY belong_id) as bn GROUP BY bn.bus;
```

### IF 语句
```shell
# case 1
SELECT
  project_name,
  repo,
  env_name,
  IF (json_extract(setting, '$.enabled') = true, '开启', '关闭')
FROM
  `table_a`
where
  repo in ('xxxx', 'yyyy');
# case 2
SELECT if(id in(
  select distinct(`table_b`.`project_id`)  from `table_b`
    where `table_b`.`category`= 'set'), '使用', '不使用')  as '状态',
       count(1)  as '数量'
  from table_c;
```

### 中文乱码问题解决
```shell
show variables like'%char%';  # 查看
set character_set_results=utf8;
set character_set_client=utf8;
set character_set_connection=utf8;
set character_set_database=utf8;
set character_set_results=utf8;
```

### 更新某列为另一列的值
```shell
UPDATE table_a SET integration_id = auto_test_id WHERE id = 102 and integration_id != auto_test_id;
```

### JSON字段处理
JSON对象
- 使用对象操作的方法进行查询：字段->'$.json属性'   
- 使用函数进行查询：json_extract(字段, '$.json属性')   
- 获取JSON数组/对象长度：JSON_LENGTH()

```shell
# 查询面料不为空的商品
select * from test where desc_attr->'$.material' is not null;
select * from test where JSON_EXTRACT(desc_attr, '$.material') is not null;

# 查询面料为纯棉的商品
select * from test where desc_attr->'$.material'='纯棉';
select * from test where JSON_EXTRACT(desc_attr, '$.material')='纯棉';

# 查询标签数量大于2的商品
select * from test where JSON_LENGTH(desc_attr->'$.tag')>2;
```
JSON数组
- 对象操作方式查询：字段->'$[0].属性'   
- 使用函数查询：JSON_CONTAINS(字段,JSON_OBJECT('json属性', '内容'))   
- 获取JSON数组/对象长度：JSON_LENGTH()   

```shell
# 查询描述属性不为空的商品
select * from test2 where JSON_LENGTH(desc_attrs) > 0;

# 查询第1项存在颜色属性的商品
select * from test2 where desc_attrs->'$[0].color' is not null;

# 查询任意项存在链接属性的商品
select * from test2 where desc_attrs->'$[*].link' is not null;

# 查询任意项存在链接等于xxx属性的商品
select * from test2 where JSON_CONTAINS(desc_attrs,JSON_OBJECT('link', 'xxx'));
```

更多案例
```shell
select * FROM table_a where JSON_TYPE(`extra_info`) = 'NULL' limit 10;  # json_length(table_a.reason) = 0;
# json为对象时，查询姿势一
select * from users where json_extract(address, '$.province') = "河北省";
# json为对象时，查询姿势二
select * from users where address -> '$.province' = "河北省";
# json为数组时，查询姿势
select * from users where address -> '$[0]'= "家";
# 根据json对象里的属性个数进行查询
select * from users where json_length(address) = 2;
# 查询JSON数组里面对象属性任意项存在指定属性的数据
select * from users where address->'$[*].city' is not null;
# 查询JSON对象存在指定属性的数据
select * from users where address->'$.tags' is not null;
```

### 正则匹配
```shell
select count(*) from table_a inner join table_b on table_a.id = table_b.color_id where table_a.version = '527' and table_a.name not REGEXP '-[0-9]{3}-[0-9]+-[0-9]+$' order by color_id desc;
```
