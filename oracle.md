#oracle
##一对多的时候就往往选择分组
分组
`select sum(sal) from emp_sal where deptno = 30`
结果：
```
  SUM(SAL)
----------
      9400
```
```
select deptno,sum(sal) from emp_bak
group by deptno
having deptno = 30
 
```
结果：
```
DEPTNO   SUM(SAL)
------ ----------
    30       9400
```
###最大值最小值
`select min(sal) from emp_bak`
```
  MIN(SAL)
----------
       800
```

`select * from emp_bak where sal=(select min(sal) from sal)`
```
EMPNO ENAME      JOB         MGR HIREDATE          SAL      COMM DEPTNO
----- ---------- --------- ----- ----------- --------- --------- ------
 7369 SMITH      CLERK      7902 1980/12/17     800.00               20

```
###约束
`ssex varchar2(3) not null ,default '男' ,check  (ssex in('男','女')),`
default和not null不能同时用

```
create table student(
sno number(3) primary key,
sname varchar2(20) not null,
ssex varchar2(3) not null,check (ssex in ('男','女')),
sbirthday date not null,
class number(5) not null
)
```
###经典查询
12、查询Sc表中至少有5名学生选修的并以3开头的课程的平均分数。
错误演示
```
1
select * from sc 
where (select avg (score) from sc where cno = 3%)
2
select avg(score) from sc 
group by cno
having cno = '3%' and count(cno)>5
```
```
select avg(score) from sc 
group by cno
having cno like '3%' and count(cno)>=5
```

自然连接（如何实现三个表连接）
```
15、查询所有学生的Sno、Cname和Score列。 (正确演示)【join using】
select sno,cname,score --字段不能有表名（student.sno）
from sc join course using(cno)
join student using(sno)

【join on】
select student.sno,course.cname,sc.score --要为字段知名表名
from sc join course on(sc.cno = course.cno)
join student on(student.sno= sc.sno)

```
----
```

select course.cname,sc.sno,sc.score from course,sc where sc.cno = course.cno ;
--等效：
select sc.sno,course.cname,sc.score 
from sc  join course on (sc.cno = course.cno)
course.cno ;
--等效：
select sc.sno,course.cname,sc.score 
from sc  join course using(cno)


```
```
(select student.sno,course.cname,sc.score 
from student natural join course,sc
并没有消除笛卡尔积
)
```
```

select student.sno,course.cname,sc.score 
from student natural join course,sc on sc.sno = student.sno,sc.cno = course.cno
命令为正确结束！

```
###批量插入
```
--错误语句：
insert into grade values(70,79,'C')
select low,upp,rank from dual from dual
union
insert into grade values(60,69,'D')
select low,upp,rank from dual from dual
union
insert into grade values(0,59,'E')
select low,upp,rank from dual from dual

--正确打开方式：
insert into grade (low,upp,rank)
select 70,79,'C' from dual 
union
select 60,69,'D'from dual
union
select 0,59,'E' from dual
```
### 不等连接  
不同表的内容，比较分组显示
```
--错误演示：
select sc.sno,sc.cno,grade.rank from sc,grade 
group by rank 
having score > low and score <upp

--正确打开方式：
select sc.sno,sc.cno,grade.rank from sc,grade  
where sc.score <grade.low and sc.score < grade.upp;


----------------------------------------------------
--显示员工的编号,姓名,工资,以及工资所对应的级别
select emp.empno,emp.ename,emp.sal,salgrade.grade
from emp,salgrade
where emp.sal <=salgrade.hisal and
emp.sal >= salgrade.losal

```

19、查询选修“3105”课程的成绩高于“109”号同学成绩的所有同学的记录。
```
select * from sc 
where score > (select score from sc where sno = 109 and cno = 3105);


|SNO|   CNO|  SCORE |
---- ----- ------
| 103 | 3245  | 86.0|
| 103 | 3105  | 92.0|
| 105 | 3105  | 88.0|
| 107 | 3105  | 91.0|
 108  3105   78.0
 101  6166   85.0
 107  6166   79.0
 108  6166   81.0

```


```
笛卡尔积//
select * from student,sc
where score > (select score from sc where sno = 109 and cno = 3105);
```
###单行和多行查询
** = 只能返回一个值而 in能返回多个值** 精准查询和模糊查询
```
--错误示范：因为sno= 后面返回的是一组数据
select * from student where sno = (select sno from sc where score > (select score from sc where sno = 109 and cno = 3105));

--正确打开方式：
select * from student where sno in (select sno from sc where score > (select score from sc where sno = 109 and cno = 3105));
```

查询Sc中选学一门以上课程的同学中分数为非最高分成绩的记录。
```
--- 查询没门的最高成绩
select * from sc 
where score in 
(select max(score) from sc group by cno);
```

###分组
```
-- 查询选修某课程的同学人数多于5人的教师姓名。
错误演示：（本质上其实是返回的所有教师名字，应该按课程号分组）
select tname from teacher where (select count(cno) from sc)>5 ;
```