# 概述
本题目的表结构大致如下图所示，具体的建表及测试数据可以参考原博主的博客。<br>

我已经连通练习过一次并记录在了为知笔记中，也借此得到了很大的锻炼。这一次进行第二次练习熟悉，也顺便录入到Github中来。<br>

另，本篇中的题目，前半部分难度稍大，后半部分难度减小。<br>

<img src="https://github.com/deng-cc/MyTraining/blob/master/pics/sql/sql_student_01.jpg?raw=true" width="600"  />


# 练习

1）查询“001”课程比“002”课程成绩高的所有学生的学号

``` stylus
select
  s1.StudentNo
from 
  score s1, 
  score s2
where 
  s1.CourseNo = 001 
  and 
  s2.CourseNo = 002 
  and 
  s1.StudentNo = s2.StudentNo
  and
  s1.score > s2.score
```
<br>

2） 查询平均成绩大于60分的同学的学号和平均成绩

``` stylus
select
  s1.StudentNo,
  AVG(s1.score)
from
  score s1
  group by s1.StudentNo
  having AVG(s1.score) > 60
```
<br>

3）查询所有同学的学号、姓名、选课数、总成绩

``` stylus
select
  stu1.studentNo,
  stu1.name,
  count(s1.CourseNo),
  sum(s1.score)
from
  score s1,
  student stu1
where
  s1.StudentNo = stu1.studentNo
  group by s1.StudentNo
```
<br>

4）查询姓“李”的老师的个数

``` stylus
select
  count(*)
from
  teacher t1
where
  t1.name like '李%'
```
<br>

5）查询没学过“叶平”老师课的同学的学号、姓名

``` stylus
select
  stu1.studentNo,
  stu1.name
from
  student stu1
where
  stu1.studentNo not in
  (
  --学过叶平老师课的学生学号
  select distinct
    s1.StudentNo
  from
    score s1,
    course c1,
    teacher t1
  where
    s1.CourseNo = c1.courseNo
    and
    c1.teacherNo = t1.teacherNo
    and
    t1.name = '叶平'
  )
```
<br>

6）查询学过“001”并且也学过编号“002”课程的同学的学号、姓名

``` stylus
select
  stu1.studentNo,
  stu1.name
from
  score s1,
  student stu1
where
  s1.StudentNo = stu1.studentNo
  and
  s1.CourseNo = 001
  and
  s1.StudentNo in
  (
  select distinct
    s2.StudentNo
  from
    score s2
  where
    s2.CourseNo = 002
  )
```
<br>

7）查询学过“叶平”老师所教的所有课的同学的学号、姓名

``` stylus
select
  s1.StudentNo,
  stu1.name
from
  score s1,
  student stu1,
  course c1,
  teacher t1
where
  s1.StudentNo = stu1.studentNo
  and
  s1.CourseNo = c1.courseNo
  and
  c1.teacherNo = t1.teacherNo
  and
  t1.name = '叶平'
  group by s1.StudentNo
  having count(c1.courseNo)=
  (
    select
      count(c2.courseNo)
    from
      course c2,
      teacher t2
    where
      c2.teacherNo = t2.teacherNo
      and
      t2.name = '叶平'
   )
```
<br>

8）查询课程编号“002”的成绩比课程编号“001”课程低的所有同学的学号、姓名（和第1题是重复的）

``` stylus
select
  s1.StudentNo,
  stu1.name
from
  score s1,
  score s2,
  student stu1
where
  s1.StudentNo = stu1.studentNo
  and
  s1.CourseNo = 001
  and
  s2.CourseNo = 002
  and
  s1.StudentNo = s2.StudentNo
  and
  s1.score > s2.score
```
<br>

9）查询有课程成绩小于60分的同学的学号、姓名

``` stylus
select distinct
  s1.StudentNo,
  stu1.name
from
  score s1,
  student stu1
where
  s1.StudentNo = stu1.studentNo
  and
  s1.score < 60
```
<br>

10）查询没有学全所有课的同学的学号、姓名

``` stylus
select
  s1.StudentNo,
  stu1.name
from
  score s1,
  student stu1
where
  s1.StudentNo = stu1.studentNo
  group by s1.StudentNo
  having count(s1.CourseNo) <
  (
  select
    count(*)
  from
    course c1
  )
```
<br>

11）查询至少有一门课与学号为“001”的同学所学相同的同学的学号和姓名

``` stylus
select distinct
  s1.StudentNo,
  stu1.name
from
  score s1,
  student stu1
where
  s1.StudentNo = stu1.studentNo
  and
  s1.CourseNo in
  (
  select
    s2.CourseNo
  from
    score s2
  where
    s2.StudentNo = 001
  )
```
<br>

12）查询至少学过学号为“001”同学所有一门课的其他同学学号和姓名（和11题几乎一样）
``` stylus
select distinct
  s1.StudentNo,
  stu1.name
from
  score s1,
  student stu1
where
  s1.StudentNo = stu1.studentNo
  and
  s1.StudentNo != 001
  and
  s1.CourseNo in
  (
  select
    s2.CourseNo
  from
    score s2
  where
    s2.StudentNo = 001
  )
```
<br>

**13）把“SC”表中“叶平”老师教的课的成绩都更改为此课程的平均成绩**

这道题有两个点需要注意：
- MySQL中不支持先select同一表中的某值，再update这个表，所以需要采用再select的方式来提取值；
- 个人理解的该题的题意是，假设叶平老师教授了A课和B课，那么课程A改为课程A的平均成绩，课程B改为课程B的平均成绩。实际上我写出的是课程A和B的成绩都改为A和B成绩的平均成绩。~~所以实际上我认为以下的写法是有违题意的，但是按照理解的题意，暂未想到合适的sql语句，故此搁浅，隔日再审。~~

``` stylus
update 
  score s1
set
  s1.score = 
  (
  select
    table1.avgScore
  from
    (
    select
      avg(s2.score) as avgScore
    from
      score s2,
      course c2,
      teacher t2
    where
      s2.CourseNo = c2.courseNo
      and
      c2.teacherNo = t2.teacherNo
      and
      t2.name = '叶平'
    )table1  
  )
  
where
  s1.CourseNo in
  (
  select
    c1.courseNo
  from
    course c1,
    teacher t1
  where
    c1.teacherNo = t1.teacherNo
    and
    t1.name = '叶平'
  )
```
以上为当初不符题意的sql写法，后来知道原来update可以采取类似多表联合的方式，于是得到了解决，具体代码如下（另，相关文章参考链接：[MySQL:把一个表中的数据按键值更新(update)到另一个表][1]），读书少害人啊！

``` stylus
UPDATE
  score s,
  (
  SELECT
    s1.CourseNo as courseNo,
    AVG(s1.score) as avgScore 
  FROM
    score s1,
    course c1,
    teacher t1
  WHERE
    s1.CourseNo = c1.CourseNo
    AND
    c1.teacherNo = t1.teacherNo
    AND
    t1.name = '叶平'
  GROUP BY s1.CourseNo
  ) as t
SET
  s.score = t.avgScore
WHERE
  s.CourseNo = t.courseNo
```
<br>

**14）查询和“002”号的同学学习的课程完全相同的其他同学学号和姓名**

这里利用了CourseNo作为主键唯一的特性，不同人的各课程相加的值也不同，如果相同，那么所学课程必定相同。
``` stylus
select
  s1.StudentNo,
  stu1.name
from
  score s1,
  student stu1
where
  s1.StudentNo = stu1.studentNo
  and
  s1.StudentNo != 2
  group by s1.StudentNo
  having sum(s1.CourseNo) =
  (
  select
    sum(s2.CourseNo)
  from
    score s2
  where
    s2.StudentNo = 2
  )
```
<br>

15）删除学习“叶平”老师课的SC表记录

``` stylus
delete from 
  score s1
where
  s1.CourseNo in
  (
  select
    c1.courseNo
  from
    course c1,
    teacher t1
  where
    c1.teacherNo = t1.teacherNo
    and
    t1.name = '叶平'
  )

```
<br>

**16）向SC表中插入一些记录，这些记录要求符合以下条件：1、没有上过编号“002”课程的同学学号；2、插入“002”号课程的平均成绩**

（本题采用插入子查询的方式，三个字段中后两个字段为常量）

``` stylus
insert into score(StudentNo, courseNo, score)
(
select
  stu1.StudentNo, 
  "002",
  (
  select
    avg(s1.score)
  from
    score s1
  where
    s1.CourseNo = 2
  ) 
from
  student stu1
where
  stu1.studentNo not in
  (
  select
    s2.StudentNo
  from
    score s2
  where
    s2.CourseNo = 2
  )
)
```
<br>

**17）按平均成绩从低到高显示所有学生的“语文”、“数学”、“英语”三门的课程成绩，按如下形式显示： 学生ID,语文,数学,英语,有效课程数,有效平均分**

（在自然连接和等值连接中，不匹配的元组信息会丢失，如果在这道题中使用，就会不完整。这里采用了相关子查询的方式。）

``` stylus
select
  s.StudentNo,
  (select s1.score from score s1 where s1.CourseNo=1 and s1.StudentNo = s.StudentNo) as "语文",
  (select s2.score from score s2 where s2.CourseNo=2 and s2.StudentNo = s.StudentNo) as "数学",
  (select s3.score from score s3 where s3.CourseNo=3 and s3.StudentNo = s.StudentNo) as "英语",
  COUNT(s.CourseNo) as "有效课程数",
  AVG(s.score) as "有效平均分"
from
  score s
  group by s.StudentNo
  order by AVG(s.score)
```
先将score s按s.StudentNo分类，则StudentNo不重复，分别提出和子查询中参与运算，如[ select s1.score from score s1 where s1.CourseNo=1 and s1.StudentNo = s.StudentNo ]中，提取s中StudentNo如1到子查询中，在子查询中找到s1.StudentNo = s.StudentNo = 1的，然后继续条件and s1.CourseNo = 1，如果没有，填为null。
<br>
<br>

18）查询各科成绩最高和最低的分：以如下形式显示：课程ID，最高分，最低分；

``` stylus
select
  s1.CourseNo as '课程ID',
  max(s1.score) as '最高分',
  min(s1.score) as '最低分'
from
  score s1
  group by s1.CourseNo
```
<br>

**19）按各科平均成绩从低到高和及格率的百分数从高到低顺序；**

``` stylus
select
  s1.CourseNo,
  avg(s1.score) as 'avgScore',
  t1.passNum / count(s1.StudentNo) as 'passRate'
from
  score s1, 
  (
  select
    s2.CourseNo,
    count(s2.StudentNo) as 'passNum'
  from
    score s2
  where
    s2.score > 60
    group by s2.CourseNo
  )t1
where
  s1.CourseNo = t1.CourseNo
  group by s1.CourseNo
  order by avgScore, passRate desc 
```
（这种解法有误，忽略了某学科全部成绩低于60的情况，信息丢失）

所以采取外连接的方式，重写如下：

``` stylus
select
  t0.CourseNo,
  t0.avgScore,
  t1.passNum,
  t0.amount,
  t1.passNum / t0.amount * 100 as 'passRate'
from
  (
  select
    s1.CourseNo,
    avg(s1.score) as 'avgScore',
    count(s1.score) as 'amount'
  from
    score s1
    group by s1.CourseNo
  )t0
  natural left outer join
  (
  select
    s2.CourseNo,
    count(s2.StudentNo) as 'passNum'
  from
    score s2
  where
    s2.score > 60
    group by s2.CourseNo
  )t1
  order by t0.avgScore, passRate desc
```
科目1所有人都没及格，你会发现这个查询结果及格数为null，但是我们想要的是，如果没有就为0，怎么办？那么我们在使用ifNull函数：

``` stylus
select
  t0.CourseNo,
  t0.avgScore,
  ifnull(t1.passNum, 0),
  t0.amount,
  ifnull(t1.passNum, 0) / t0.amount * 100 as 'passRate'
from
  (
  select
    s1.CourseNo,
    avg(s1.score) as 'avgScore',
    count(s1.score) as 'amount'
  from
    score s1
    group by s1.CourseNo
  )t0
  natural left outer join
  (
  select
    s2.CourseNo,
    count(s2.StudentNo) as 'passNum'
  from
    score s2
  where
    s2.score > 60
    group by s2.CourseNo
  )t1
  order by t0.avgScore, passRate desc
```
这里，提一下MYSQL中ISNULL和IFNULL的区别：
ISNULL(expr)：if expr is NULL, returns 1, else returns 0;
IFNULL(expr1, expr2)：if expr1 is not NULL, returns expr1, else returns expr2.
<br>
<br>

20）查询不同老师所教不同课程平均分从高到低显示

``` stylus
select
  t1.name,
  c1.courseNo,
  c1.name,
  avg(s1.score)
from
  score s1,
  course c1,
  teacher t1
where
  s1.CourseNo = c1.courseNo
  and
  c1.teacherNo = t1.teacherNo
  group by s1.CourseNo
  order by avg(s1.score) desc
```
<br>

**21）统计列印各科成绩,各分数段人数: 课程ID,课程名称,(100-85),(85-70),(70-60),( 低于60)**

这里用到了MySQL中 case when then else end 用法，命令如其名，详见题目SQL语句。

``` stylus
select
  s1.CourseNo,
  c1.name,
  sum(case when s1.score between 85 and 100 then 1 else 0 end) as "85-100",
  sum(case when s1.score between 70 and 85 then 1 else 0 end) as "70-85",
  sum(case when s1.score between 60 and 70 then 1 else 0 end) as "60-70",
  sum(case when s1.score < 60 then 1 else 0 end) as "<60"
from
  score s1,
  course c1
where
  s1.CourseNo = c1.courseNo
  group by s1.CourseNo
```
<br>

**22）查询各科成绩前三名的记录:(不考虑成绩并列情况)**

这里会巧妙利用到相关子查询。

``` stylus
select
  *
from
  score s1
where
  s1.score in
  (
  select
    s2.score
  from
    score s2
  where
    s2.CourseNo = s1.CourseNo
    order by s2.score desc
    limit 0, 3
  )
```
如果每次提出的该行中的分数，在该分数对应学科的前三名分数当中，那么符合条件，提取出来。
但是，在MySQL4.1中子查询是不能使用LIMIT的，手册中也明确指明 This version of MySQL doesn’t yet support ‘LIMIT & IN/ALL/ANY/SOME subquery’。
所以这个方法暂时待定，换另一个方法：

``` stylus
select
  *
from
  score s
where
  (
  select
    count(*)
  from
    score s1
  where
    s1.CourseNo = s.CourseNo
    and
    s1.score > s.score
  ) < 3
order by s.CourseNo
```
这个方法的思路是，需要提取的行，该行对应的分数对应的学科中，比它高的分数必须小于3个 
（即假如自己是第3，那么在该学科所有分数中，比自己高的只有第一和第二共2个）
但是，这个方法对于并列的成绩，会全部输出。
<br>
<br>

23）查询每门课程被选修的学生数

``` stylus
select
  s1.CourseNo,
  count(s1.StudentNo)
from
  score s1
  group by s1.CourseNo

```
<br>

24）查询出只选修了一门课程的全部学生的学号和姓名

``` stylus
select
  s1.StudentNo,
  stu1.name
from
  score s1,
  student stu1
where
  s1.StudentNo = stu1.studentNo
  group by s1.StudentNo
  having count(s1.CourseNo) = 1
```
<br>

25）查询男生、女生的人数

``` stylus
select
  stu1.sex,
  count(stu1.studentNo)
from
  student stu1
  group by stu1.sex

```
<br>

**26）查询同名同姓学生名单，并统计同名人数**

这道题告诉我，凡是不要想得太复杂...

``` stylus
select
  stu1.name,
  count(stu1.name)
from
  student stu1
  group by stu1.name
```
<br>

**27）查询1991年出生的学生名单**

不同数据库的时间函数不一样，这里使用的是mysql；<br>
CURDATE() 或CURRENT_DATE()  获取当前日期，按格式"YYYY-MM-DD"返回；<br>
YEAR(date)  返回date对应的年份，范围是1970-2069

``` stylus
select
  stu1.name,
  stu1.age
from
  student stu1
where
  YEAR(CURRENT_DATE())-stu1.age = 1991
```
<br>

28）查询每门课程的平均成绩，结果按平均成绩升序排列

``` stylus
select
  s1.CourseNo,
  avg(s1.score)
from
  score s1
  group by s1.CourseNo
  order by avg(s1.score)
```
<br>

29）查询平均成绩大于85的所有学生的学号、姓名和平均成绩

``` stylus
select
  stu1.studentNo,
  stu1.name,
  avg(s1.score)
from
  score s1,
  student stu1
where
  s1.StudentNo = stu1.studentNo
  group by s1.StudentNo
  having avg(s1.score)>85
```
<br>

30）查询课程名称为“数学”，且分数低于60的学生姓名和分数

``` stylus
select
  stu1.name,
  s1.score
from
  score s1,
  course c1,
  student stu1
where
  s1.StudentNo = stu1.studentNo
  and
  s1.CourseNo = c1.courseNo
  and
  c1.name = "数学"
  and
  s1.score < 60
```
<br>

31）查询所有学生的选课情况

``` stylus
select
  stu1.name,
  c1.name
from
  score s1,
  student stu1,
  course c1
where
  s1.StudentNo = stu1.studentNo
  and
  s1.CourseNo = c1.courseNo
  order by stu1.studentNo
```
<br>

32）查询任何一门课程成绩在70分以上的姓名、课程名称和分数

``` stylus
select
  stu1.name,
  c1.name,
  s1.score
from
  score s1,
  course c1,
  student stu1
where
  s1.CourseNo = c1.courseNo
  and
  s1.StudentNo = stu1.studentNo
  and
  s1.score > 70
```
<br>

33）查询不及格的课程，并按课程号从大到小排列

``` stylus
select distinct
  s1.CourseNo
from
  score s1
where
  s1.score < 60
  order by s1.CourseNo desc
```
<br>

34）查询课程编号为003且课程成绩在80分以上的学生的学号和姓名

``` stylus
select
  s1.StudentNo,
  stu1.name,
  s1.score
from
  score s1,
  student stu1
where
  s1.StudentNo = stu1.studentNo
  and
  s1.CourseNo = 3
  and
  s1.score > 80
```
<br>

**35）求选了课程的学生人数**

``` stylus
select
count(*)
from
(
select
  *
from
  score s1
  group by s1.StudentNo
) table1
```

不够简洁，优化如下：

``` stylus
select
  count(distinct s1.StudentNo)
from
  score s1
```
<br>

**36）查询选修“杨艳”老师所授课程的学生中，成绩最高的学生姓名及其成绩**

使用MySQL的limit函数：

``` stylus
select
  stu1.name,
  s1.score
from
  score s1,
  course c1,
  teacher t1,
  student stu1
where
  s1.CourseNo = c1.courseNo
  and
  s1.StudentNo = stu1.studentNo
  and
  c1.teacherNo = t1.teacherNo
  and
  t1.name = '杨艳'
  order by s1.score desc
  limit 0, 1
```
<br>

37）查询各个课程及相应的选修人数

``` stylus
select
  c1.courseNo,
  c1.name,
  count(*)
from
  score s1,
  course c1
where
  s1.CourseNo = c1.courseNo
  group by s1.CourseNo
```
<br>

**38）查询不同课程但成绩相同的学生的学号、课程号、学生成绩**

``` stylus
select distinct
  s1.StudentNo,
  s1.CourseNo,
  s1.score
from
  score s1,
  score s2
where
  s1.score = s2.score 
  and
  s1.CourseNo <> s2.CourseNo
  order by s1.score
```
<br>

**39）查询每门课程成绩最好的前两名**

这里同第22题：查询各科成绩前三名的记录

``` stylus
select
  stu1.name,
  s.CourseNo,
  s.score
from
  score s,
  student stu1
where
  (
  select
    count(*)
  from
    score s1
  where
    s.CourseNo = s1.CourseNo
    and
    s.score < s1.score 
  ) < 2
  and
  s.StudentNo = stu1.studentNo
  order by s.CourseNo
```
<br>

40）统计每门课程的学生选修人数（超过5人的课程才统计）要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

``` stylus
select
  s1.CourseNo,
  count(s1.StudentNo)
from
  score s1
  group by s1.CourseNo
  having count(s1.StudentNo)>5
  order by count(s1.StudentNo) desc, s1.CourseNo
```
<br>

41）检索至少选修两门课程的学生学号

``` stylus
select
  s1.StudentNo
from
  score s1
  group by s1.StudentNo
  having count(s1.CourseNo) >= 2
```
<br>

**42）查询全部学生都选修的课程的课程号和课程名**

思路：全部学生都选修了的课程，即：score表按课程分组，课程中的学生数量和总学生数量相同的元组，即为全部学生选修的课程

``` stylus
select
  s1.CourseNo,
  c1.name
from
  score s1,
  course c1
where
  s1.CourseNo = c1.courseNo
  group by s1.CourseNo
  having count(s1.StudentNo)=
  (
  select
    count(*)
  from
    student stu1
  )
```
<br>

43）查询没学过“叶平”老师讲授的任一门课程的学生姓名

``` stylus
select
  stu1.name
from
  student stu1
where
  stu1.StudentNo not in
  (
  select distinct
    s1.StudentNo
  from
    score s1,
    course c1,
    teacher t1
  where
    s1.CourseNo = c1.courseNo
    and
    c1.teacherNo = t1.teacherNo
    and
    t1.name = '叶平'
  )
```
<br>

44）查询两门以上不及格课程的同学的学号及其平均成绩

``` stylus
select
  t1.StudentNo,
  t1.avgScore
from
(
select
  s2.StudentNo,
  avg(s2.score) as avgScore
from
  score s2
  group by s2.StudentNo
) t1,
(
select
  s1.StudentNo
from
  score s1
where
  s1.score < 60
  group by s1.StudentNo
  having count(s1.CourseNo)>=2
) t2
where
  t1.StudentNo = t2.StudentNo
```
<br>

45）检索“004”课程分数小于60，按分数降序排列的同学学号

``` stylus
select
  s1.StudentNo,
  s1.score
from
  score s1
where
  s1.CourseNo = 4
  and
  s1.score < 60
  order by s1.score desc
```
<br>

46）删除“002”同学的“001”课程的成绩

``` stylus
enter code heredelete from
  score s1
where
  s1.StudentNo = 2
  and
  s1.CourseNo = 1 
```
<br>
<br>

END


  [1]: http://www.aichengxu.com/mysql/21657.htm