---
title: pgsql 模糊查询
date: 2022-09-12 16:34:25
categories:
- [数据仓库]
top_img: http://tva1.sinaimg.cn/large/006UraCCly1h63yex45pej32yo1o0b2j.jpg
cover: http://tva1.sinaimg.cn/large/006UraCCly1h63yex45pej32yo1o0b2j.jpg
---  
# pgsql 模糊查询  
## 前言
+ 参考资料：  
 https://blog.csdn.net/qq_32838955/article/details/105466577  
       https://zhuanlan.zhihu.com/p/60010253  
 https://www.cnblogs.com/alianbog/p/5657115.html  
 https://www.postgresql.org/docs/9.6/functions-textsearch.html  
 gin索引 https://razeen.me/posts/pg-like-index-optimize/
+ 测试表 
```text

create table test as (
    name text,
    name info text
)

```
## like/not like  
+ like 支持模糊匹配 区分大小写,%为通配符 下划线 _ 代表任意一个字符
```text

-- 匹配name中包含zhang且前面只有一个字符的数据
select * from test where name like '_zhang%'
```
## ~ POSIX正则表达式
+ ~ + POSIX正则表达式 查询符合条件的行，或者~ + %模糊匹配 与like有相同作用
```text

-- 查询name中以 zhang 为开头的数据
select * from test where name ~'^zhang' 
-- 查询name中包含 zhang 的数据
select * from test where name ~'%zhang%'
```
## similar to 正则表达式
+ similar to + (正则表达式)实现多条件匹配
```text

-- 查询name中姓赵或钱或孙或李的数据
select * from test where name similar to '(赵|钱|孙|李)%'
```
## tsvector
+ pgsql 自带tsvector 文本检索类型，提高文本检索效率，tsvector类型产生一个文档（以优化全文检索形式）和tsquery类型用于查询检索，但是str_to_tsvector 不适用于英文
```text

-- 去除文本中所有的中英文符号
create function clear_punctuation(text)
  returns text
immutable
strict
language sql
as $$
select regexp_replace($1,
	'[\ |\~|\`|\!|\@|\#|\$|\%|\^|\&|\*|\(|\)|\-|\_|\+|\=|\||\\|\[|\]|\{|\}|\;|\:|\"|\''|\,|\<|\.|\>|\/|\?|\：|\。|\；|\，|\：|\“|\”|\（|\）|\、|\？|\《|\》]'
	,'','g');
$$;

-- 将字符串转换为tsvector, 每连续两个字符作为一个词
drop function if exists str_to_tsvector(text);
create or replace function str_to_tsvector(text)
returns tsvector
as $$
	declare
		v_count integer;
		v_txt text;
		v_txts text[];
		v_result tsvector;
	begin
		v_txt := clear_punctuation($1);
		--数组大小为字符数量-1
		v_count := length(v_txt)-1;
		if( v_count < 1 ) then
			raise exception '输入参数("%")去除标点符号后至少需要2个字符',$1;
		end if;
		for i in 1..v_count loop
			v_txts := array_append(v_txts, substring(v_txt,i,2));
		end loop;
		--tsvector类型要求去除重复并排序
		with cte1 as(
			select f from unnest(v_txts) as f group by f
		),cte2 as(
			select f from cte1 order by f
		)select array_to_tsvector(array_agg(f)) into v_result from cte2;
		return v_result;
	end;
$$ language plpgsql strict immutable;

-- 将字符串转换为tsquery
drop function if exists str_to_tsquery(text,boolean);
create or replace function str_to_tsquery(text,boolean default true)
returns tsquery
as $$
	declare
		v_count integer;
		v_txt text;
		v_txts text[];
		v_result tsquery;
	begin
		v_txt := clear_punctuation($1);
		--数组大小为字符数量-1
		v_count := length(v_txt)-1;
		if( v_count < 1 ) then
			raise exception '输入参数("%")去除标点符号后至少需要2个字符',$1;
		end if;
		for i in 1..v_count loop
			v_txts := array_append(v_txts, substring(v_txt,i,2));
		end loop;
		--tsquery类型要求去除重复并排序
		with cte1 as(
			select f from unnest(v_txts) as f group by f
		),cte2 as(
			select f from cte1 order by f
		)select string_agg(f, (case when $2 then '&' else '|' end ) )::tsquery into v_result from cte2;
		return v_result;
	end;
$$ language plpgsql strict immutable;

-- 准备
insert into ts_test(name,name_info_ts) 
select name,str_to_tsvector(name_info) as name_info_ts from test;
-- 创建gin索引
create index ts_test_name_info_ts_gin_index on ts_test using gin (name_info_ts);
-- 查询
select * from ts_test where name_info_ts @@ (str_to_tsquery('杭州'))
```
