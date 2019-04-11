# 大数据

## 一、数据仓库

### 1.1 拉链表

![1548066942504](C:\Users\suntiansheng2\AppData\Roaming\Typora\typora-user-images\1548066942504.png)

注意：一般start_date和end_date为同一天，离线任务一天跑一次，一天归档一次，这样能保证都是最新的一条记录。

![img](file:///c:\users\suntiansheng2\documents\jddongdong\jimenterprise\suntiansheng\image\f21da027bfe97423.png)



## 二、HIVE

### 2.1 Hive简介

1.Hive是一个数据仓库

2.Hive基于Hadoop

总结为一句话：hive是基于hadoop的数据仓库。

### 2.2 Hive 的安装















### 1.2 常用的窗口函数

https://blog.csdn.net/scgaliguodong123_/article/details/60135385

https://blog.csdn.net/qq_31807385/article/details/84777349

取top n：

row_number()

```sql
select * ,row_number() over(partition by sale_ord_id order by last_update_tm desc) as rn from gdm.gdm_m04_ord_sum where dp='ACTIVE' and to_date(last_update_tm)>=sysdate(-7) ) t where rn=1
```





rank

dense_ran()



### 1.3 常用排序

ORDER BY 全局排序，只有一个Reduce任务

SORT BY 只在本机做排序



### 1.4 常用命令



### 1.5 HIVE SQL

1. ***注释***

   -- 

   ("#","/**/")不能用

2. ***if函数***

语法: if(boolean testCondition, T valueTrue, T valueFalseOrNull)

返回值: T

说明:  当条件testCondition为TRUE时，返回valueTrue；否则返回valueFalseOrNull

```sql
// if语句相当于三目运算符，注意：等价于用单等号'='和双等号'=='都可以
hive> select if(1<2,100,200);
OK
_c0
100
Time taken: 0.127 seconds, Fetched: 1 row(s)

// 单等号'='
hive> select if(1=1,100,200);
OK
_c0
100
Time taken: 0.111 seconds, Fetched: 1 row(s)

// 双等号'=='
hive> select if(1==1,100,200);
OK
_c0
100
Time taken: 0.274 seconds, Fetched: 1 row(s)

```

3. ***非空查找函数: COALESCE***

语法: COALESCE(T v1, T v2, …)

返回值: T

说明:  返回参数中的第一个非空值(不为NULL)；如果所有值都为NULL，那么返回NULL

```sql
hive> select coalesce(null,'Z');
OK
_c0
Z
Time taken: 0.189 seconds, Fetched: 1 row(s)
```

4. ***CASE a WHEN b THEN b' \[WHEN c THEN c'][ELSE d] END***

   注意：和java中的switch case差不多

   如果a等于b b'

   如果a等于c c‘

   如果a既不等于a又不等于b d

   ```sql
   hive> select case 1 when 1 then 1 end;
   OK
   _c0
   1
   Time taken: 0.074 seconds, Fetched: 1 row(s)
   
   
   hive> select case 5 when 1 then 1 when 2 then 2 else 3 end;
   OK
   _c0
   3
   Time taken: 0.096 seconds, Fetched: 1 row(s)
   
   ```

   

5. **CASE WHEN a THEN a' [WHEN c THEN c']\* [ELSE d] END**

   相当于多层if...else 嵌套

   ```sql
   hive> select case when 90<60 then 'unqualified' when 90>60 then 'qualified' end;
   OK
   _c0
   qualified
   Time taken: 0.206 seconds, Fetched: 1 row(s)
   
   ```


6. hive中提供了两种针对json数据格式解析的函数，即get_json_object（…）与json_tuple(…)

7. ***Hive的一些参数设置***

   

8. ***Hive多表插入***

   Hive支持多表插入，可以在同一个查询中使用多个insert子句，这样的好处是我们只需要扫描一遍源表就可以生成多个不相交的输出！

   语法

   ```sql
   -- from ......    insert ......    insert ......
   ```

9. ***Hive排除查询字段***

   ```sql
   select `(name|id|pwd)?+.+` from tableName;
   ```

10. ***Hive regexp_replace函数***

    ```sql
    -- 	ware_ids字段：[{"skuid":"100273","skunum":1}]
    -- regexp_replace(string INITIAL_STRING, string PATTERN, string REPLACEMENT)
    select id
               ,get_json_object( regexp_replace(regexp_replace(ware_ids,'\\[','') ,'\\]',''),'$.skuid') skuid
               ,get_json_object( regexp_replace(regexp_replace(ware_ids,'\\[','') ,'\\]',''),'$.skunum')  skunum 
          from fdm.fdm_refundfc_refund_major_chain 
          where dp='ACTIVE' and ware_ids <> '' limit 10
    ```

    

11. ***统计某个字段大于2条的记录***

    ```sql
    -- 第一种，嵌套查询
    select
    	*
    from
    	(
    		select
    			ref_id,count(ref_id) ref_id_count
    		from
    			fdm.fdm_refundfc_refund_detail_chain
    		where
    			dp = 'ACTIVE'
    			and pay_type in(31, 32, 33, 34, 35, 36, 39, 38)
    		group by
    			ref_id
    	)b
    where
    	b.ref_id_count > 2;
    	
    -- 第二种，group by ... having ...
    
    select
    	ref_id,
    	count(ref_id) ref_id_count
    from
    	fdm.fdm_refundfc_refund_detail_chain
    where
    	dp = 'ACTIVE'
    	and pay_type in(31, 32, 33, 34, 35, 36, 39, 38)
    group by
    	ref_id
    having
    	ref_id_count > 2
    ```

    > 参考：
    >
    > #### [为什么mysql having的条件表达式可以直接使用select后的别名？](https://www.cnblogs.com/leisurelylicht/p/wei-shen-memysql-having-de-tiao-jian-biao-da-shi-k.html)
    >
    > SQL语句的语法顺序：
    >
    > ```
    > FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> DISTINCT -> UNION -> ORDER BY
    > ```
    >
    > 因此一般不能在having condition中使用select list中的alias。
    >
    > 但是mysql对此作了扩展。在mysql 5.7.5之前的版本，ONLY_FULL_GROUP_BY sql mode默认不开启。在5.7.5或之后的版本默认开启。
    >
    > 如果ONLY_FULL_GROUP_BY sql mode不开启，那么mysql对标准SQL的扩展可以生效：
    >
    > 1. 允许在select list、having condition和order by list中使用没有出现在group by list中的字段。此时mysql会随机选择没有出现在group by list中的字段的值。效果和使用ANY_VALUE()是相同的。
    > 2. 允许在having condition中使用select list中的alias



12. group by 后面多个字段

      

13. 设计表结构

14. 

