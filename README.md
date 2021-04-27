# 个人笔记
本人在工作中的一些小的发现和感悟，事无巨细都记录下来，方便自己查看
目前手头的活偏向于报表开发，暂时没有太深入的东西，就是写Hive Sql

# Hive SQL(报表开发)

基本理解：

hadoop生态圈里面的一个组件。说白了就是解析器，解析Hive SQL并转换成相应的MR代码，达到与写代码相同的效果，降低Hadoop分析的使用门槛(倒是有操作系统的味道在里面了)。

一般会用HBase整合Hive使用，然而直接使用的话也很麻烦，所以公司里面进行报表开发的时候一般会选择使用图形化界面来进行，一来效果是一样的，二来也不会有人因为操作不熟悉乱敲指令导致出什么幺蛾子。



工作场景：Hive整合DBeaver使用。

DBeaver等数据库图形化工具可以直接连接上Hive，直接将Hive当做传统数据库使用进行一些开发非常方便，总而言之只要会写SQL就行了，然后注意一些比如HiveSQL过滤条件中不支持子查询这种细节就行了。



## 1.Hive执行计划 explain

在SQL开头声明 explain；可以查看Hive的执行计划

![1618197277454](HelloHive.assets/1618197277454.png)

因为测试环境比较简陋，而且估计因为账号权限的原因，只要SQL中一加配置就无法运行，只有执行计划可以正常看。

## 2.Oracle Sql优化为Hive Sql 

1--66--22.45S--原Sql

select a.id from table_1 a where not exists (select 'X'
          from table_2 b where b.contno = a.otherno and b.moneytype in ('YFLQ', 'EFLQ'))

--2--62--21.33S--用left join 优化

select a.id from t1 a left join table_2 b on  on b.contno = a.otherno and b.moneytype in ('YFLQ', 'EFLQ')
where b.contno is null

有个好玩的地方，如果原SQL是Exists的时候，使用这种方法会有一些差别：

使用Exists判断的结果在Hive中运行完了会有一个去重的效果，而使用Left join 优化以后得到的结果不会去重(如果需要的话，手动加一个去重感觉反而不划算，不如不优化)，这个差别让我感觉很有意思，不知道在Oracle中会不会有这种差异。(对于Exists，Hive有专门的特殊用法，所以这个问题很难发生)

## 3.Oracle和Hive的一些函数差异(Hive为主)

### 3.1.字符串操作函数

substr(a,1,10);-->Oracle中用法一样，起始位置注意一下，Hive中是从一开始的

regexp_replace(a,'^[b]');-->注意正则表达式用的是JAVA的正则

instr(a,'b');-->Oracle中带四个参数：instr(原字段，要匹配的字符串，起始位置，第几次匹配)，很方便

concat(a,b,c......);-->在Oracle中只能连接两个字符串(concat(a,b))，而用:`||`可以连接多个字符串达到与Hive相同的效果(a||b||c...)。

concat_ws(',',collect_set(t1.name));列转行函数，将所有的name通过逗号连接在一起

目前用到的就是这几个，在Hive中，substr需要三个参数，输出一个截取后的字符串

#### 截取函数substr

substr(字段，起始截取位置，截取长度)

substr(regexp_replace(l.standbyflag6,'^(.*?\\//.*?/.*?/.*?/)',''),1,10)

#### 替换函数regexp_replace

regexp_replace(原字符串,'正则表达式','替换后显示的字符串')带三参返回一个经过替换操作的字符串

#### 匹配位置函数instr

instr(原字段,'要匹配的字符串')

#### concat_ws(',',collect_set(t1.name))

需要两个参数，第一个是分隔符，默认应该是空格，没有试过

collect_set()，还有collect_list(),将一列转换为一个列表，但是set有去重的效果

例：篇幅原因，看前五条就行了

原本数据的样子：

![1619060368811](HelloHive.assets/1619060368811.png)

这是用List：结果成了一行，有重复

![1619060435689](HelloHive.assets/1619060435689.png)

这是用Set：结果也是一行，但是没有重复

![1619060498583](HelloHive.assets/1619060498583.png)

出于好奇，我又跑了一下collect_set(),不多逼逼，结果如下：
![1619060653147](HelloHive.assets/1619060653147.png)



### 3.2 时间函数

date();-->Oracle 中用法为to_date(日期字符串，格式)

但是Hive中用date转换的结果不能直接用于计算(日期的加减，取月份之类的)，我找了一个麻烦一点的代替，可以用于运算的函数方法：

from_unixtime(unix_timestamp(日期字符串，字段格式)，目标格式)

后面有碰到一种：

date_format(to_date(),'yyyy-MM-dd')

### 3.3 NVL

看到网上有人四月份发帖，说Hive不支持NVL，遂写个demo验证下：
![1619159516699](HelloHive.assets/1619159516699.png)

结果为一列。

![1619159748297](HelloHive.assets/1619159748297.png)

说明生效了，没有NVL不能用的说法。

然后是另一个帖子说的，NVL函数前后结果类型必须一样，为此我找了一个类型为string的函数，结果如下：

![1619160630520](HelloHive.assets/1619160630520.png)

然而找了好久没在集群的库中找到原本为date类型的函数，无法验证，甚是遗憾。

## 4.Exist和not Exist

`Exist`可以用`left semi join` 代替（执行计划上看一模一样）

`not exist` 可以用` t1 left join t2 on t1.id=t2.id where t2.id is null `代替,记住一定要用`where`不然结果不准确(有待深究)

### 4.1  Left Semi Join

Left semi join 可以完全代替Exists，但是同时也有很多限制
1.Left Semi Join的所有过滤条件必须都加在on连接条件之后，否则无法运行

2.Left Semi Join只支持简单的过滤，如果涉及到非等值连接还有or连接的判断的话就无法使用，建议拆开用 Union all

## 5.函数嵌套的修改(遇到一个复杂的记一个)

```sql
nvl(nvl((select codename
               from ldcode
              where codetype = 'paymode'
                and code = (select c.paymode from ljaget c where c.otherno = d.edorconfno and rownum = 1 and c.enteraccdate is not null)
                and rownum = 1
         ),
                (select codename
               from ldcode
              where codetype = 'paymode'
                and code = (select c.paymode from ljaget c where c.otherno = d.edorconfno and rownum = 1 and c.enteraccdate is null)
                and rownum = 1))
     ,
             (select codename
                from ldcode
               where codetype = 'edorgetpayform'
                 and code = d.payform
                 and rownum = 1))
                 
explain
select               
nvl(nvl(ldcode1.code_name,ldcode2.code_name),ldcode3.code_name)
from ms_ods_p05.lpedorapp d
left join ms_ods_p05.ljaget c1 on c1.otherno = d.edorconfno and c1.enteraccdate is not null
left join ms_ods_p05.ljaget c2 on c2.otherno = d.edorconfno and c2.enteraccdate is null
left join ms_ods_p06.ldcode ldcode1 on ldcode1.code=c1.paymode and ldcode1.code_type = 'paymode' 
left join ms_ods_p06.ldcode ldcode2 on ldcode2.code=c2.paymode and ldcode2.code_type = 'paymode' 
left join ms_ods_p06.ldcode ldcode3 on ldcode3.code = d.payform and ldcode3.code_type = 'edorgetpayform'
```

## 6.比较头疼的一些暂时没解决的逻辑问题(持续更新)

### 6.1 过滤条件Exists中的逻辑问题

Hive 子查询不支持非等值连接，t1.id < t2.id 这种连接条件无法在子查询中使用

and过滤条件中用exists判断两个子查询or的结果，问题是第二个子查询中带了非等值连接导致问题解决不了：

```sql
and exists
    (select 1
     from lmriskapp
     where riskcode = a.riskcode
     and riskperiod = 'L')
     and ((appflag = '1' and exists
         (select 1
          from lccontstate
          where polno = a.polno
          and statetype = 'Available'
          and state = '0'
          and enddate is null)) or
          (exists
           (select 1
            from lccontstate
            where polno = a.polno
            and statetype = 'Available'
            and state = '0'
            and startdate >= to_date('${P1}', 'YYYY-mm-dd')
            and enddate <= add_months(cvalidate, 24))))
```

**难点：**

1.子查询判断中用or连接，这个可以用`union all`解决

2.上半部分分开以后没问题，下半部分单独运行都会报错

```sql
(exists
     (select 1
      from lccontstate
      where polno = a.polno
      and statetype = 'Available'
      and state = '0'
      and startdate >= to_date('${P1}', 'YYYY-mm-dd')
      and enddate <= add_months(cvalidate, 24)))
```

`and enddate <= add_months(cvalidate, 24)` 这个条件是非等值连接，hive 本身不支持非等值连接

最终解决办法：

经过分析得知，`enddate <= add_months(cvalidate, 24)` 过滤的是到期两年后，也就是过期的订单。

然后我直接

```sql
with temp as (select lc.polno from lccontstate lc,主表 a where enddate > add_months(a.cvalidate, 24))
先拿到所有没过期的订单的订单号
```

然后通过

```sql
and polno=temp.polno
```

留下的都是没过期的，实现了要实现的功能。

### 6.2 麻烦的函数嵌套问题

原：

```sql
nvl((select  decode(nvl(LTRIM(substr(e.impartparammodle, 1,INSTR(e.impartparammodle, '/', 1, 1) - 1),'0123456789'),'1'),'1',substr(e.impartparammodle, 1,
                               INSTR(e.impartparammodle, '/', 1, 1) - 1)||'元','0')
                   from LCCustomerImpart e
                  where e.CustomerNoType = '0'
                    and e.CustomerNo =a.appntno
                    and e.ContNo = a.contno
                    and e.impartver ='102'
                    AND E.IMPARTCODE ='20B'
                    and rownum = 1),(select  decode(nvl(LTRIM(substr(e.impartparammodle, 1,INSTR(e.impartparammodle, '/', 1, 1) - 1),'0123456789'),'1'),'1',substr(e.impartparammodle, 1,
                INSTR(e.impartparammodle, '/', 1, 1) - 1)||'万','0')
                   from LCCustomerImpart e
                  where e.CustomerNoType = '0'
                    and e.CustomerNo = a.appntno
                    and e.ContNo =a.contno
                    and e.impartver = 'A06'
                    AND E.IMPARTCODE = 'A0534'
                    and rownum = 1)) 年收入,
```

修改后：

```Sql
nvl( t1.num1 , t2.num2 ) `年收入`,

left join (select  case nvl(regexp_replace(substr(e.impartparammodle, 1, instr(e.impartparammodle,'/') ),'^\\d*',''),'1') when '1' then concat(substr(e.impartparammodle, 1,
           INSTR(e.impartparammodle, '/') - 1),'元') else '0' end  as num1,CustomerNo,ContNo
				   from ms_ods_p06.LCCustomerImpart e
				  where e.CustomerNoType = '0'
				 	and e.impartver ='102'
					AND E.IMPARTCODE ='20B' group by e.impartparammodle,CustomerNo,ContNo) t1 on t1.CustomerNo =a.appntno and t1.ContNo = a.contno
left join (select  case nvl(regexp_replace(substr(e.impartparammodle, 1,INSTR(e.impartparammodle, '/') ),'^\\d*',''),'1') when '1' then concat(substr(e.impartparammodle, 1,
                INSTR(e.impartparammodle, '/') ),'万') else '0' end as num2,CustomerNo,ContNo
                   from ms_ods_p06.LCCustomerImpart e
                  where e.CustomerNoType = '0'
                    and e.impartver = 'A06'
                    AND E.IMPARTCODE = 'A0534' group by e.impartparammodle，CustomerNo,ContNo) t2 on t2.CustomerNo =a.appntno and t2.ContNo = a.contno


```

## 7. 92写法和99写法带来的差异

```sql
--Hive 中 ,连接和 inner join 连接到底有什么区别
--92
explain
select a.contno,b.contno from ms_ods_p06.lpedoritem a,ms_ods_p08.lccont b  where b.contno = a.contno
--99
explain
select a.contno,b.contno from ms_ods_p06.lpedoritem a inner join ms_ods_p08.lccont b on b.contno = a.contno
```

分别看执行计划，前面都一模一样，在第二个stage才出现了差异：

92：

![1619159175816](HelloHive.assets/1619159175816.png)

而99：

![1619159110547](HelloHive.assets/1619159110547.png)

很清楚地看到92语法在select操作前多了一步过滤操作，导致后面进行的操作影响的行数比较少。但是由于过滤操作本身带来的性能差异就难说了，数据量不同的情况下性能优劣真不好说。

## 8.去重留一

Hive 版：

select edorstate from ( select edorstate,ROW_NUMBER() over(partition by edorstate order by edorno desc) as num from ms_ods_p06.lpedoritem ) temp where temp.num=1;

Row_number()函数可以跟开窗，好用

![1619511376529](HelloHive.assets/1619511376529.png)

漂亮的去重留一。
