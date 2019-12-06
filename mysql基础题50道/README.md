# Mysql基本语法练习
>  题目来源 : [SQL 50题](https://zhuanlan.zhihu.com/p/43289968)

## 表结构简介

* Student(s_id, s_name, s_birth, s_sex) // 学生编号、 学生姓名、 学生生日、 学生性别
* Course(c_id, c_name, t_id)            // 课程编号、 课程名称、 教师编号
* Teacher(t_id, t_name)                // 教师编号、 教师姓名
* Score(s_id, c_id, s_score)           // 学生编号、 课程编号、 分数

![](./pic/关系图.png)

##  1.查询课程编号为“01”的课程比“02”的课程成绩高的所有学生的学号（重点）

    思路：
        1. 需要一张表 NewTable(s_id, c_id1_score, c_id2_score)
        2. 找出c_id = 01的学生 select s_id, s_score from score where c_id = '01' 
        3. 找出c_id = 02的学生 select s_id, s_score from score where c_id = '02'
        4. 两张表连接，用学生的编号来关联 inner join on student t1.s_id = t2.s_id
        5. 第四步得到的NewTable, 查询where t1.s_score > t2.s_score 得到新的表
        6. 再把新表和Student关联引入s_name, inner join Student on s_id

```
select id_t.s_id, s_name from
(
select t1.s_id from
(
(select s_id, s_score as s1 from score where c_id = '01') as t1
inner join
(select s_id, s_score as s2 from score where c_id = '02') as t2
on 
t1.s_id = t2.s_id) where s1 > s2) as id_t
inner join Student  as name_t
on name_t.s_id = id_t.s_id;
```
        +------+--------+                                                                                                                                            
        | s_id | s_name |                                                                                                                                            
        +------+--------+                                                                                                                                            
        | 02   | 钱电   |                                                                                                                                            
        | 04   | 李云   |                                                                                                                                            
        +------+--------+ 

## 2. 查询平均成绩大于60分的学生的学号和平均成绩（简单，第二道重点）
    
    思路：
        1. 对Score表group by s_id.
        2. 对每组的s_score 求avg

```
select s_id, avg(s_score) as avg_score
    from score
group by s_id having avg(s_score) > 60;
```
        +------+-----------+                                                                                                                                         
        | s_id | avg_score |                                                                                                                                         
        +------+-----------+                                                                                                                                         
        | 01   |   89.6667 |                                                                                                                                         
        | 02   |   70.0000 |                                                                                                                                         
        | 03   |   80.0000 |                                                                                                                                         
        | 05   |   81.5000 |                                                                                                                                         
        | 07   |   93.5000 |                                                                                                                                         
        +------+-----------+ 

***group by 通常和聚合函数一起使用***

***having 用在group之后的筛选， where用在group之前***

## 3. 查询所有学生的学号、姓名、选课数、总成绩（不重要）
        思路：
                1. 把student表和score表关联起来.
                2. 通过group by s_id 对学生分组， 求出每组的科目数count, 总成绩 sum
        
```
select st.s_id as id, st.s_name as name, count(sc.c_id) as count, sum(case when sc.s_score is NULL then 0 else sc.s_score end) as sum
from
(
        student as st
        left join
        score as sc
        using(s_id)
) group by 
s_id;

```
        +----+------+-------+------+                                                                                                                                 
        | id | name | count | sum  |                                                                                                                                 
        +----+------+-------+------+                                                                                                                                 
        | 01 | 赵雷 |     3 |  269 |                                                                                                                                 
        | 02 | 钱电 |     3 |  210 |                                                                                                                                 
        | 03 | 孙风 |     3 |  240 |                                                                                                                                 
        | 04 | 李云 |     3 |  100 |                                                                                                                                 
        | 05 | 周梅 |     2 |  163 |                                                                                                                                 
        | 06 | 吴兰 |     2 |   65 |                                                                                                                                 
        | 07 | 郑竹 |     2 |  187 |                                                                                                                                 
        | 08 | 王菊 |     0 |    0 |                                                                                                                                 
        +----+------+-------+------+ 

***通过case when 过滤一些特殊的情况如NULL***

## 4. 查询姓“猴”的老师的个数（不重要）
        思路：
                1. 在teacher表中用 t_name like '猴%'
                2. count(t_id)

```
select count(t_id) as teacher猴
from teacher
where t_name like '猴%'
```

        +-----------+                                                                                                                                                
        | teacher猴 |                                                                                                                                                
        +-----------+                                                                                                                                                
        |         0 |                                                                                                                                                
        +-----------+  

***like 用法模糊查找 %表示任意字符***

## 5. 查询没学过“张三”老师课的学生的学号、姓名（重点）

        思路：
                1. 先把teacher表和course表关联起来 using(t_id)
                2. 逆向思维找出学过张三老师课的 where t_name = '张三'
                3. 使用not in


```
select  st.s_id, st.s_name
from student as st
where st.s_id not in
(
        select so.s_id
        from score as so
        where so.c_id in
        (select co.c_id
        from 
        (
                course as co
                inner join 
                teacher as te
                using(t_id)
        )
        where te.t_name = '张三') 
)
```
        +------+--------+                                                                                                                                            
        | s_id | s_name |                                                                                                                                            
        +------+--------+                                                                                                                                            
        | 06   | 吴兰   |                                                                                                                                            
        | 08   | 王菊   |                                                                                                                                            
        +------+--------+  

***使用了in 和not in***

## 6. 查询学过“张三”老师所教的所有课的同学的学号、姓名（重点）

        思路：
                1. 上一题改为in

```
select  st.s_id, st.s_name
from student as st
where st.s_id  in
(
        select so.s_id
        from score as so
        where so.c_id in
        (select co.c_id
        from 
        (
                course as co
                inner join 
                teacher as te
                using(t_id)
        )
        where te.t_name = '张三') 
)
```
        +------+--------+                                                                                                                                            
        | s_id | s_name |                                                                                                                                            
        +------+--------+                                                                                                                                            
        | 01   | 赵雷   |                                                                                                                                            
        | 02   | 钱电   |                                                                                                                                            
        | 03   | 孙风   |                                                                                                                                            
        | 04   | 李云   |                                                                                                                                            
        | 05   | 周梅   |                                                                                                                                            
        | 07   | 郑竹   |                                                                                                                                            
        +------+--------+  

## 7. 查询学过编号为“01”的课程并且也学过编号为“02”的课程的学生的学号、姓名（重点）
        思路：
                1. 找出学过01的学生 一个表 学过02的学生一个表
                2. 两个表inner join

```
select  st.s_id as id, st.s_name as name
from student as st
where st.s_id in
(select s01.s_id
from 
(
        (select * from score where c_id = '01') as s01
        inner join
        (select * from score where c_id = '02') as s02
        using(s_id)
)
)
```

        +----+------+                                                                                                                                                
        | id | name |                                                                                                                                                
        +----+------+                                                                                                                                                
        | 01 | 赵雷 |                                                                                                                                                
        | 02 | 钱电 |                                                                                                                                                
        | 03 | 孙风 |                                                                                                                                                
        | 04 | 李云 |                                                                                                                                                
        | 05 | 周梅 |                                                                                                                                                
        +----+------+  

## 8. 查询课程编号为“02”的总成绩（不重点）
        思路：
                1. 在score表中使用sum()

```
select sum(s_score) 
from score
where 
c_id = '02'
```

        +--------------+                                                                                                                                             
        | sum(s_score) |                                                                                                                                             
        +--------------+                                                                                                                                             
        |          436 |                                                                                                                                             
        +--------------+ 

## 9. 查询所有课程成绩小于60分的学生的学号、姓名

        思路：
                1. 在score表中找出 < 60的 s_id
                2. student 查找 在上面不及格表中的人in

```
select st.s_id as id, st.s_name as name
from
student as st 
where
st.s_id in
(
        select  sc.s_id
        from score as sc
        where sc.s_score < 60
);
```

        +----+------+                                                                                                                                                
        | id | name |                                                                                                                                                
        +----+------+                                                                                                                                                
        | 04 | 李云 |                                                                                                                                                
        | 06 | 吴兰 |                                                                                                                                                
        +----+------+  

## 10. 查询没有学全所有课的学生的学号、姓名(重点)

        思路：
                1. 找出有几门课 count(c_id)
                2. 找出学生分组后课程数小于上面的count

```
select st.s_id as id, st.s_name as name
from 
(
        student as st
        inner join
        score as sc
        using(s_id)
)
group by st.s_id having count(c_id) < 
(
        select count(distinct c_id) from course
) 
```

        +----+------+                                                                                                                                                
        | id | name |                                                                                                                                                
        +----+------+                                                                                                                                                
        | 05 | 周梅 |                                                                                                                                                
        | 06 | 吴兰 |                                                                                                                                                
        | 07 | 郑竹 |                                                                                                                                                
        +----+------+  

***计算数量时候考虑去重distinct***

## 11. 查询至少有一门课与学号为“01”的学生所学课程相同的学生的学号和姓名（重点）

        思路：
                1. 找出01号学生所学的课
                2. 用in

```
select distinct st.s_id as id, st.s_name as name
from 
(
        student as st
        inner join
        score as sc
        using(s_id)
) where sc.c_id in
(
        select distinct sc2.c_id
        from 
        (
                score as sc2
        )
        where sc2.s_id = '01' 
);
```
        +----+------+                                                                                                                                                
        | id | name |                                                                                                                                                
        +----+------+                                                                                                                                                
        | 01 | 赵雷 |                                                                                                                                                
        | 02 | 钱电 |                                                                                                                                                
        | 03 | 孙风 |                                                                                                                                                
        | 04 | 李云 |                                                                                                                                                
        | 05 | 周梅 |                                                                                                                                                
        | 06 | 吴兰 |                                                                                                                                                
        | 07 | 郑竹 |                                                                                                                                                
        +----+------+  

## 12. 查询和“01”号同学所学课程完全相同的其他同学的学号(重点)

        思路：
                1. 找出01同学学的课
                2. 从sorce表中找出所有 in　01课程的行，并且 s_id <>  '01'
                3. group by s_id， 找出count = 01同学课的人

```
select  st.s_id as s_id, st.s_name as name
from
student as st
where st.s_id in
(
        select  sc1.s_id as s_id
        from score as sc1
        where 
        sc1.c_id in 
        (
                select sc2.c_id as c_id
                from score as sc2
                where sc2.s_id = '01'
        ) and sc1.s_id <> '01'
        group by sc1.s_id having count(sc1.c_id)  = (
                select count(sc3.c_id) from score as sc3 where sc3.s_id = '01'
        )
)
```

        +------+------+                                                              
        | s_id | name |                                                              
        +------+------+                                                              
        | 02   | 钱电 |                                                              
        | 03   | 孙风 |                                                              
        | 04   | 李云 |                                                              
        +------+------+  

## 13. 查询没学过"张三"老师讲授的任一门课程的学生姓名 和47题一样（重点，能做出来）
        思路：
                1. 找出张三老师的课程表
                2. 找出上过任意一门的同学 in
                3. student not in 表2


```
select  st.s_id as id, st.s_name as name
from 
student as st
where 
st.s_id not in (
        select sc1.s_id
        from 
        score as sc1
        where sc1.c_id in (
                select  sc2.c_id
                from (
                        score as sc2
                        left join
                        course as co
                        using(c_id)
                ) 
                where co.t_id = 
                 (
                        select te.t_id  from teacher as te where te.t_name = '张三'
                )

        )
)
```

        +----+------+                                                                
        | id | name |                                                                
        +----+------+                                                                
        | 06 | 吴兰 |                                                                
        | 08 | 王菊 |                                                                
        +----+------+ 

## 15. 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩（重点）

        思路：
                1. where筛选 < 60
                2. group by having count >= 2
                3. avg()

```
select a.s_id as id, b.s_name as name, a.avg_score as avg
from
(select sc.s_id, avg(s_score) as avg_score  
from
score as sc
where sc.s_score < 60
group by sc.s_id
having count(sc.s_score) >=2) as a
left join
student as b
using(s_id)
```
        +----+------+---------+                                                      
        | id | name | avg     |                                                      
        +----+------+---------+                                                      
        | 04 | 李云 | 33.3333 |                                                      
        | 06 | 吴兰 | 32.5000 |                                                      
        +----+------+---------+ 

## 16. 检索"01"课程分数小于60，按分数降序排列的学生信息（和34题重复，不重点）

        思路：
                1. order by

```
select sc.s_id, st.s_name, sc.s_score
from
score as sc 
inner join
student as st
using(s_id)
where
sc.c_id = '01' and  sc.s_score < 60
order by sc.s_score desc
```
        +------+--------+---------+                                                  
        | s_id | s_name | s_score |                                                  
        +------+--------+---------+                                                  
        | 04   | 李云   |      50 |                                                  
        | 06   | 吴兰   |      31 |                                                  
        +------+--------+---------+

## 17. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩(重重点与35一样)

        思路：
                1. 分组配合聚合函数使用


```
select  
st.s_name, 
max(case when sc.c_id = '01' then sc.s_score else null end) 'math',
max(case when sc.c_id = '02' then sc.s_score else null end) 'chinese',
max(case when sc.c_id = '03' then sc.s_score else null end) 'English',
avg(sc.s_score)
from
(student as st
inner join
score as sc
using(s_id)
)
group by st.s_id
order by avg(sc.s_score) desc
```
        +--------+------+---------+---------+-----------------+                      
        | s_name | math | chinese | English | avg(sc.s_score) |                      
        +--------+------+---------+---------+-----------------+                      
        | 郑竹   | NULL |      89 |      98 |         93.5000 |                      
        | 赵雷   |   80 |      90 |      99 |         89.6667 |                      
        | 周梅   |   76 |      87 |    NULL |         81.5000 |                      
        | 孙风   |   80 |      80 |      80 |         80.0000 |                      
        | 钱电   |   70 |      60 |      80 |         70.0000 |                      
        | 李云   |   50 |      30 |      20 |         33.3333 |                      
        | 吴兰   |   31 |    NULL |      34 |         32.5000 |                      
        +--------+------+---------+---------+-----------------+ 

## 18 查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率--及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90 (超级重点)

        思路：

```
select sc.c_id as id, co.c_name as name, max(sc.s_score) as max, min(sc.s_score) as min, avg(sc.s_score) as avg, 
avg(case when sc.s_score < 60 then 1 else 0 end) as  failure_rate,
avg(case when sc.s_score > 60 and sc.s_score <= 70 then 1 else 0 end) as  just_rate,
avg(case when sc.s_score > 70 and sc.s_score <= 80 then 1 else 0 end) as  mid_rate,
avg(case when sc.s_score > 80 and sc.s_score <= 90 then 1 else 0 end) as  good_rate,
avg(case when sc.s_score > 90 and sc.s_score <= 100 then 1 else 0 end)as  excellent_rate
from
(
        course as co
        inner join
        score as sc
        using(c_id)
)
group by sc.c_id

```

        +----+------+------+------+---------+--------------+-----------+----------+-----------+----------------+                                                     
        | id | name | max  | min  | avg     | failure_rate | just_rate | mid_rate | good_rate | excellent_rate |                                                     
        +----+------+------+------+---------+--------------+-----------+----------+-----------+----------------+                                                     
        | 01 | 语文 |   80 |   31 | 64.5000 |       0.3333 |    0.1667 |   0.5000 |    0.0000 |         0.0000 |                                                     
        | 02 | 数学 |   90 |   30 | 72.6667 |       0.1667 |    0.0000 |   0.1667 |    0.5000 |         0.0000 |                                                     
        | 03 | 英语 |   99 |   20 | 68.5000 |       0.3333 |    0.0000 |   0.3333 |    0.0000 |         0.3333 |                                                     
        +----+------+------+------+---------+--------------+-----------+----------+-----------+----------------+ 

***avg、sum函数里面用case when ，加1 可以统计符合条件的个数***

## 19. 按各科成绩进行排序，并显示排名(重点row_number)row_number(）over （order by 列）

        思路：
                1. 使用row_number() over (order by)

```
select sc.c_id, co.c_name, st.s_name as name, row_number() over(order by s_score desc)
from 
(
        (score as sc
        inner join
        course as co
        using(c_id))
        left join
        student as st
        using(s_id)

)
group by c_id;

```

![](https://pic2.zhimg.com/80/v2-561472090beb2924c13e28e1f511488d_hd.jpg)

## 20. 查询学生的总成绩并进行排名（不重点）
        思路：
                1. order by sum()

```
select st.s_id as id, st.s_name as name, sum(sc.s_score) as sum
from (
        score as sc
        left join
        student as st
        using(s_id)
)
group by sc.s_id
order by sum(sc.s_score) desc
```

        +------+------+------+                                                                                                                                       
        | id   | name | sum  |                                                                                                                                       
        +------+------+------+                                                                                                                                       
        | 01   | 赵雷 |  269 |                                                                                                                                       
        | 03   | 孙风 |  240 |                                                                                                                                       
        | 02   | 钱电 |  210 |                                                                                                                                       
        | 07   | 郑竹 |  187 |                                                                                                                                       
        | 05   | 周梅 |  163 |                                                                                                                                       
        | 04   | 李云 |  100 |                                                                                                                                       
        | 06   | 吴兰 |   65 |                                                                                                                                       
        +------+------+------+  

## 21. 查询不同老师所教不同课程平均分从高到低显示(不重点)

        思路： 
                1. teacher表和 course表和 score表join
                2. order by avg

```
select te.t_id as id, te.t_name as name, avg(sc.s_score)
from (
        teacher as te
        inner join
        course as co
        using(t_id)
        inner join
        score as sc
        using(c_id)
)
group by te.t_id
order by avg(sc.s_score) desc;
```
        +----+------+-----------------+                                                                                                                              
        | id | name | avg(sc.s_score) |                                                                                                                              
        +----+------+-----------------+                                                                                                                              
        | 01 | 张三 |         72.6667 |                                                                                                                              
        | 03 | 王五 |         68.5000 |                                                                                                                              
        | 02 | 李四 |         64.5000 |                                                                                                                              
        +----+------+-----------------+  

## 22. 查询所有课程的成绩第2名到第3名的学生信息及该课程成绩（重要 25类似）

        思路：
                1. row_number () over (partition by 分组列 order by 排序列)

```
select *
from (
        select row_number () over(partition by c_id order by s_core desc) m
        from(
                score sc inner join student st on sc.s_id
        ) 
) as a
where m in (2, 3)
```

![](https://pic3.zhimg.com/80/v2-2a865bc462b0bf5283e9ca9e30f6218e_hd.jpg)

## 23. 使用分段[100-85],[85-70],[70-60],[<60]来统计各科成绩，分别统计各分数段人数：课程ID和课程名称(重点和18题类似)

        思路：
                1. sum(case when then end)

```
select 
sc.c_id, co.c_name, 
sum(case when sc.s_score > 0 and sc.s_score <= 60 then 1 else 0 end) as D,
sum(case when sc.s_score > 60 and sc.s_score <= 70 then 1 else 0 end) as C,
sum(case when sc.s_score > 70 and sc.s_score <= 85 then 1 else 0 end) as B,
sum(case when sc.s_score > 85 and sc.s_score <= 100 then 1 else 0 end) as A
from (
        score as sc
        left join
        course as co
        using(c_id)
)
group by sc.c_id
```

        +------+--------+------+------+------+------+                                                                                                                
        | c_id | c_name | D    | C    | B    | A    |                                                                                                                
        +------+--------+------+------+------+------+                                                                                                                
        | 01   | 语文   |    2 |    1 |    3 |    0 |                                                                                                                
        | 02   | 数学   |    2 |    0 |    1 |    3 |                                                                                                                
        | 03   | 英语   |    2 |    0 |    2 |    2 |                                                                                                                
        +------+--------+------+------+------+------+  

## 24. 查询学生平均成绩及其名次（同19题，重点）
        思路：
                1. row_number () over (oder by)
        
```
select s_id, svg(s_score), row_number () over (order by avg(s_score) desc)
from score
group by s_id
```

我的MySQL不支持所以就不跑了
![](https://pic2.zhimg.com/80/v2-2af82357f928dad031a28d29bb1e2731_hd.jpg)

## 25. 查询各科成绩前三名的记录（不考虑成绩并列情况）（重点 与22题类似）
        思路：
                1. row_number () over(partition by c_id order by ...)

![](https://pic4.zhimg.com/80/v2-c69c9697a961e9594c97718356218cc7_hd.jpg)

## 26. 查询每门课程被选修的学生数(不重点)
        思路：
                1. group by 课程 ，然后count

```
select sc.c_id, count(sc.s_id) as num
from  score as sc
group by  sc.c_id
```

        +------+-----+                                                                                                                                               
        | c_id | num |                                                                                                                                               
        +------+-----+                                                                                                                                               
        | 01   |   6 |                                                                                                                                               
        | 02   |   6 |                                                                                                                                               
        | 03   |   6 |                                                                                                                                               
        +------+-----+ 

## 27.  查询出只有两门课程的全部学生的学号和姓名(不重点)

        思路：
                1. group by => count = 2

```
select st.s_id as id, st.s_name as name
from 
(
        student as st
        right join
        score as sc
        using(s_id)
)
group by sc.s_id having count(sc.c_id) = 2;
```

        +------+------+                                                                                                                                              
        | id   | name |                                                                                                                                              
        +------+------+                                                                                                                                              
        | 05   | 周梅 |                                                                                                                                              
        | 06   | 吴兰 |                                                                                                                                              
        | 07   | 郑竹 |                                                                                                                                              
        +------+------+  

## 28. 查询男生、女生人数(不重点)
        思路：
                1. group by s_sex

```
select s_sex, count(s_sex)
from student 
group by s_sex;
```

        +-------+--------------+                                                                                                                                     
        | s_sex | count(s_sex) |                                                                                                                                     
        +-------+--------------+                                                                                                                                     
        | 女    |            4 |                                                                                                                                     
        | 男    |            4 |                                                                                                                                     
        +-------+--------------+ 

## 29. 查询名字中含有"风"字的学生信息（不重点）
        思路：
                1. like '%风%'
        
```
select  *
from student
where  s_name like '%风%';
```

        +------+--------+------------+-------+                                                                                                                       
        | s_id | s_name | s_birth    | s_sex |                                                                                                                       
        +------+--------+------------+-------+                                                                                                                       
        | 03   | 孙风   | 1990-05-20 | 男    |                                                                                                                       
        +------+--------+------------+-------+  

## 30. 查询1990年出生的学生名单（重点year）
        思路：
                1. year(date_time) 函数的使用

```
select * 
from  student
where year(s_birth) = 1990;
```

        +------+--------+------------+-------+                                                                                                                       
        | s_id | s_name | s_birth    | s_sex |                                                                                                                       
        +------+--------+------------+-------+                                                                                                                       
        | 01   | 赵雷   | 1990-01-01 | 男    |                                                                                                                       
        | 02   | 钱电   | 1990-12-21 | 男    |                                                                                                                       
        | 03   | 孙风   | 1990-05-20 | 男    |                                                                                                                       
        | 04   | 李云   | 1990-08-06 | 男    |                                                                                                                       
        | 08   | 王菊   | 1990-01-20 | 女    |                                                                                                                       
        +------+--------+------------+-------+  

![时间相关的函数](https://pic4.zhimg.com/80/v2-97bda8cf385872fa49737b5453664c8b_hd.jpg)

## 32. 查询平均成绩大于等于85的所有学生的学号、姓名和平均成绩（不重要）

        思路：
                1. group by having avg() > 85

```
select  st.s_id as id, st.s_name as name, avg(sc.s_score)
from (
        student as st
        right join
        score as sc
        using(s_id)
)
group by sc.s_id having avg(sc.s_score) > 85
```

        +------+------+-----------------+                                                                                                                            
        | id   | name | avg(sc.s_score) |                                                                                                                            
        +------+------+-----------------+                                                                                                                            
        | 01   | 赵雷 |         89.6667 |                                                                                                                            
        | 07   | 郑竹 |         93.5000 |                                                                                                                            
        +------+------+-----------------+  


## 33. 查询每门课程的平均成绩，结果按平均成绩升序排序，平均成绩相同时，按课程号降序排列（不重要）
        思路： 
                1. 两列排序的方法，先排的放在前面

```
select sc.c_id as id , co.c_name as name, avg(sc.s_score)
from (
        score as sc
        left join
        course as co
        using(c_id)
)
group by sc.c_id
order by avg(sc.s_score), sc.c_id desc;
```
        +----+------+-----------------+                                                                                                                              
        | id | name | avg(sc.s_score) |                                                                                                                              
        +----+------+-----------------+                                                                                                                              
        | 01 | 语文 |         64.5000 |                                                                                                                              
        | 03 | 英语 |         68.5000 |                                                                                                                              
        | 02 | 数学 |         72.6667 |                                                                                                                              
        +----+------+-----------------+  
```
```

## 34. 查询课程名称为"数学"，且分数低于60的学生姓名和分数（不重点）

        思路：
                1. and

```
select st.s_name as name, sc.s_score as math_score
from (
        score as sc
        left join
        course as co
        using(c_id)
        left join
        student as st
        using(s_id)
)
where co.c_name = '数学' and sc.s_score < 60;
```
        +------+------------+                                                                                                                                        
        | name | math_score |                                                                                                                                        
        +------+------------+                                                                                                                                        
        | 李云 |         30 |                                                                                                                                        
        +------+------------+  

## 35. 查询所有学生的课程及分数情况（重点）

        思路：
                1. 还是用case when (如果不知道有多少种课，这个答案也不完整)

```
select  sc.s_id as id, st.s_name as name,
max(case when co.c_name = '语文' then sc.s_score else null end) as '语文',
max(case when co.c_name = '数学' then sc.s_score else null end) as '数学',
max(case when co.c_name = '英语' then sc.s_score else null end) as '英语'
from (
        student as st
        right join
        score as sc
        using(s_id)
        left join
        course as co
        using(c_id)
)
group by sc.s_id;
```

        +----+------+------+------+------+                                                                                                                           
        | id | name | 语文 | 数学 | 英语 |                                                                                                                           
        +----+------+------+------+------+                                                                                                                           
        | 01 | 赵雷 |   80 |   90 |   99 |                                                                                                                           
        | 02 | 钱电 |   70 |   60 |   80 |                                                                                                                           
        | 03 | 孙风 |   80 |   80 |   80 |                                                                                                                           
        | 04 | 李云 |   50 |   30 |   20 |                                                                                                                           
        | 05 | 周梅 |   76 |   87 | NULL |                                                                                                                           
        | 06 | 吴兰 |   31 | NULL |   34 |                                                                                                                           
        | 07 | 郑竹 | NULL |   89 |   98 |                                                                                                                           
        +----+------+------+------+------+ 

## 36. 查询任何一门课程成绩在70分以上的姓名、课程名称和分数（重点）

        思路：
                1. 别用group
```
select st.s_name as name, co.c_name as course, sc.s_score as score
from (
        score as sc
        left join
        student as st
        using(s_id)
        left join
        course as co
        using(c_id)
)
where sc.s_score > 70;
```

        +------+--------+-------+                                                                                                                                    
        | name | course | score |                                                                                                                                    
        +------+--------+-------+                                                                                                                                    
        | 赵雷 | 语文   |    80 |                                                                                                                                    
        | 赵雷 | 数学   |    90 |                                                                                                                                    
        | 赵雷 | 英语   |    99 |                                                                                                                                    
        | 钱电 | 英语   |    80 |                                                                                                                                    
        | 孙风 | 语文   |    80 |                                                                                                                                    
        | 孙风 | 数学   |    80 |                                                                                                                                    
        | 孙风 | 英语   |    80 |                                                                                                                                    
        | 周梅 | 语文   |    76 |                                                                                                                                    
        | 周梅 | 数学   |    87 |                                                                                                                                    
        | 郑竹 | 数学   |    89 |                                                                                                                                    
        | 郑竹 | 英语   |    98 |                                                                                                                                    
        +------+--------+-------+ 

## 37. 查询不及格的课程并按课程号从大到小排列(不重点)

        思路：
                1. 先查询再排序

```
select sc.c_id as course_id, co.c_name as course_name
from (
        score as sc
        left join
        course as co
        using(c_id)
)
where sc.s_score < 60
order by c_id desc;
```

        +-----------+-------------+                                                                                                                                  
        | course_id | course_name |                                                                                                                                  
        +-----------+-------------+                                                                                                                                  
        | 03        | 英语        |                                                                                                                                  
        | 03        | 英语        |                                                                                                                                  
        | 02        | 数学        |                                                                                                                                  
        | 01        | 语文        |                                                                                                                                  
        | 01        | 语文        |                                                                                                                                  
        +-----------+-------------+ 

## 38. 查询课程编号为03且课程成绩在80分以上的学生的学号和姓名（不重要）

        思路：
                1. where and

```
select  st.s_id as id, st.s_name as name
from (
        student as st
        right join
        score as sc
        using(s_id)
)
where sc.c_id = '03' and sc.s_score > 80;
```

        +------+------+                                                                                                                                              
        | id   | name |                                                                                                                                              
        +------+------+                                                                                                                                              
        | 01   | 赵雷 |                                                                                                                                              
        | 07   | 郑竹 |                                                                                                                                              
        +------+------+ 

## 39. 求每门课程的学生人数（不重要）

        思路：
                1. count

```
select sc.c_id, co.c_name, count(sc.c_id)
from (
        score as sc
        left join
        course as co
        using(c_id)
)
group by sc.c_id;
```

        +------+--------+----------------+                                                                                                                           
        | c_id | c_name | count(sc.c_id) |                                                                                                                           
        +------+--------+----------------+                                                                                                                           
        | 01   | 语文   |              6 |                                                                                                                           
        | 02   | 数学   |              6 |                                                                                                                           
        | 03   | 英语   |              6 |                                                                                                                           
        +------+--------+----------------+ 

## 40. 查询选修“张三”老师所授课程的学生中成绩最高的学生姓名及其成绩

        思路：
                1. 先排序，然后用limit函数 limit(启示位置， 截取个数)

```
select st.s_name as name, sc.s_score as score
from (
        teacher as te
        right join
        course as co
        using(t_id)
        right join
        score as sc
        using(c_id)
        right join
        student as st
        using(s_id)
)
where te.t_name = '张三' 
order by sc.s_score limit 0, 1;
```

        +------+-------+                                                                                                                                             
        | name | score |                                                                                                                                             
        +------+-------+                                                                                                                                             
        | 李云 |    30 |                                                                                                                                             
        +------+-------+ 

## 41. 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩 （重点）

        思路：
                1. 用成绩来分组得到重复 的成绩
                2. 用where得到 与重复成绩有关的同学
                3. 对这些同学分组，如果一组中有重复的成绩，那他就是对的人。。。

```
select sc1.s_id, sc1.c_id, sc1.s_score 
from score as sc1
(select * from
where  sc1.s_score in 
(
        select sc.s_score
        from score as sc
        group by sc.s_score having count(sc.s_id) > 1
)
group by sc1.s_id having count(sc1.s_score) <= 1) as a
```

```
select distinct s1.s_id, s1.c_id, s1.s_score 
from
(
        score as s1
        inner join
        score as s2
        using(s_id)
)
where s1.s_score = s2.s_score and s1.c_id <> s2.c_id
```
        +------+------+---------+                                                                                                                                    
        | s_id | c_id | s_score |                                                                                                                                    
        +------+------+---------+                                                                                                                                    
        | 03   | 01   |      80 |                                                                                                                                    
        | 03   | 02   |      80 |                                                                                                                                    
        | 03   | 03   |      80 |                                                                                                                                    
        +------+------+---------+  
        
## 43.统计每门课程的学生选修人数（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列（不重要）

        思路：
                1. count 和 order by的使用
   
```
select sc.c_id, count(sc.c_id)
from score as sc
group by sc.c_id having count(sc.c_id) > 5
order by count(sc.c_id) desc, c_id
```
        +------+----------------+                                                                                                                                    
        | c_id | count(sc.c_id) |                                                                                                                                    
        +------+----------------+                                                                                                                                    
        | 01   |              6 |                                                                                                                                    
        | 02   |              6 |                                                                                                                                    
        | 03   |              6 |                                                                                                                                    
        +------+----------------+  

## 44. 检索至少选修两门课程的学生学号（不重要）
        思路：
                1. 同上

```
select s_id, count(c_id)
from score
group by s_id
having count(c_id) >=2
```
        +------+-------------+                                                                                                                                       
        | s_id | count(c_id) |                                                                                                                                       
        +------+-------------+                                                                                                                                       
        | 01   |           3 |                                                                                                                                       
        | 02   |           3 |                                                                                                                                       
        | 03   |           3 |                                                                                                                                       
        | 04   |           3 |                                                                                                                                       
        | 05   |           2 |                                                                                                                                       
        | 06   |           2 |                                                                                                                                       
        | 07   |           2 |                                                                                                                                       
        +------+-------------+ 

## 45. 查询选修了全部课程的学生信息

        思路：
                1. count 相等

```
select * 
from student 
where s_id in
(
        select s_id from score 
        group by s_id
        having count(c_id) = (select count(*) from course)
)
```

        +------+--------+------------+-------+                                                                                                                       
        | s_id | s_name | s_birth    | s_sex |                                                                                                                       
        +------+--------+------------+-------+                                                                                                                       
        | 01   | 赵雷   | 1990-01-01 | 男    |                                                                                                                       
        | 02   | 钱电   | 1990-12-21 | 男    |                                                                                                                       
        | 03   | 孙风   | 1990-05-20 | 男    |                                                                                                                       
        | 04   | 李云   | 1990-08-06 | 男    |                                                                                                                       
        +------+--------+------------+-------+   


## 测试数据
1. 学生表
```
CREATE TABLE `Student`(
        `s_id` VARCHAR(20),
        `s_name` VARCHAR(20) NOT NULL DEFAULT '',
        `s_birth` VARCHAR(20) NOT NULL DEFAULT '',
        `s_sex` VARCHAR(10) NOT NULL DEFAULT '',
        PRIMARY KEY(`s_id`)
        );
```

2. 课程表
```
CREATE TABLE `Course`(
        `c_id` VARCHAR(20),
        `c_name` VARCHAR(20) NOT NULL DEFAULT '',
        `t_id` VARCHAR(20) NOT NULL,
        PRIMARY KEY(`c_id`)
        )
```

3. 教师表
```
CREATE TABLE `Teacher`(
        `t_id` VARCHAR(20),
        `t_name` VARCHAR(20) NOT NULL DEFAULT '',
        PRIMARY KEY(`t_id`)
        );
```

4. 成绩表
```
CREATE TABLE `Score`(
        `s_id` VARCHAR(20),
        `c_id` VARCHAR(20),
        `s_score` INT(3),
        PRIMARY KEY(`s_id`,`c_id`)
        );
```

5. 插入数据
```
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('08' , '王菊' , '1990-01-20' , '女');
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');
insert into Score values('01' , '01' , 80);
insert into Score values('01' , '02' , 90);
insert into Score values('01' , '03' , 99);
insert into Score values('02' , '01' , 70);
insert into Score values('02' , '02' , 60);
insert into Score values('02' , '03' , 80);
insert into Score values('03' , '01' , 80);
insert into Score values('03' , '02' , 80);
insert into Score values('03' , '03' , 80);
insert into Score values('04' , '01' , 50);
insert into Score values('04' , '02' , 30);
insert into Score values('04' , '03' , 20);
insert into Score values('05' , '01' , 76);
insert into Score values('05' , '02' , 87);
insert into Score values('06' , '01' , 31);
insert into Score values('06' , '03' , 34);
insert into Score values('07' , '02' , 89);
insert into Score values('07' , '03' , 98);
```
