## 1. 取得每个部门最高薪水的人员名称
### 步骤:
第一步：求出每个部门的最高薪水（max指令） 

``` 
select # 我们最终要得到什么变量 
e.deptno,max(e.sal) as maxsal 
from # 数据来源于哪个表 
emp as e # 把表重命名为e 
group by # 按e.deptno排序并分组数据。这导致对每个e.deptno而不是整个表计算maxsal一次。 
e.deptno; 
``` 

第二步：将以上查询的结果当成一个临时表t(deptno,maxsal) 

``` 
select // 我们要得到每个部门最高薪水的人员名称，因此有部门deptno、名称ename、最高薪水maxsal 
e.deptno,e.ename,t.maxsal,e.sal 
from 
(select 
e.deptno,max(e.sal) as maxsal 
from 
emp as e 
group by 
e.deptno) as t // 将第一步查询的结果当成一个临时表t(deptno,maxsal) 
join // 把表t和表emp连接，怎么连？把deptno匹配起来 
emp as e 
on 
t.deptno = e.deptno 
where // where用于过滤，按条件过滤出我们要的结果 
t.maxsal = e.sal // 具体条件为：当t中已经得到的最高薪水maxsal等于部门员工表e中的薪水，把联立表中的该条记录（该行）提取出来 
order by // 最后用order by指令把符合条件的记录按照部门deptno升序排好序 
e.deptno; 
``` 

## 2. 哪些人的薪水在部门平均薪水之上 
### 步骤： 

第一步：求出每个部门的平均薪水（avg指令） 

``` 
select 
e.deptno,avg(e.sal) as avgsal 
from 
emp as e 
group by // 和上面是一样的逻辑，但是还是要说一下group by指令， GROUP BY 子句指示MySQL进行数据分组，然后对每个组而不是 
整个结果集进行avg。 
e.deptno; 
``` 

第二步：将以上查询的结果当成临时表t(deptno,avgsal) 
``` 
select 
t.deptno,e.ename 
from 
(select 
e.deptno,avg(e.sal) as avgsal 
from 
emp as e 
group by 
e.deptno) as t 
join 
emp as e 
on 
t.deptno = e.deptno 
where // 逻辑和第一个案例是一样的，只不过过滤条件变为选出工资大于平均工资的记录（行） 
e.sal > t.avgsal; 
``` 

## 3.获得平均薪水最高的部门的部门编号 

### 第一种方案： 
步骤： 

第一步：求出每个部门平均薪水 

``` 
select 
e.deptno,avg(e.sal) as avgsal 
from 
emp as e 
group by 
e.deptno; 
``` 

第二步：对平均薪水进行降序排序并取第一条记录 
``` 
select 
e.deptno,avg(e.sal) as avgsal 
from 
emp as e 
group by 
e.deptno 
order by 
avgsal 
DESC LIMIT 1; // desc是descend， 降序的意思，而Limit子句可以被用于强制 SELECT 语句返回指定的记录数。Limit n,m，参数n必须是一个整数常量。如果给定n和m两个参数，第一个参数n指定第一个返回记录行的偏移量，第二个参数m指定返回记录行的最大数目。 
``` 

### 第二种方案： 
步骤： 

第一步：求出每个部门平均薪水 

``` 
select 
e.deptno,avg(e.sal) as avgsal 
from 
emp as e 
group by 
e.deptno; 
``` 

第二步：把这个平均薪水表封装成临时表t，并找出最高的平均工资数额 
``` 
select 
max(t.avgsal) as maxAvgsal 
from 
(select 
e.deptno,avg(e.sal) as avgsal 
from 
emp as e 
group by 
e.deptno) as t 
``` 
第三步：把平均工资等于maxAvgsal的分组找出来 
``` 
select 
e.deptno,avg(e.sal) as avgsal 
from 
emp as e 
group by 
e.deptno 
having // having子句与while子句很类似，只不过having子句是过滤分组，而while是过滤行 
avgsal = (select // 把平均工资等于第二步求出的最高平均工资对应的那个分组过滤出来 
max(t.avgsal) as maxAvgsal 
from 
(select 
e.deptno,avg(e.sal) as avgsal 
from 
emp as e 
group by 
e.deptno) as t); 
``` 

## 7. 求平均薪水的等级最低的部门的部门名称 

第一步：求出部门的平均薪水，部门编号+名称+各部门平均薪水（avg指令） 
``` 
select // 最终我要得到部门编号，部门名称和各部门的平均工资（注意了，部门的平均工资还要借助group by指令获得） 
e.deptno,d.dname,avg(e.sal) as avgsal 
from 
emp as e 
join 
dept as d 
on 
e.deptno = d.deptno 
group by // 这里要注意了，我们是对部门和部门名称进行分组聚集 
e.deptno,d.dname; 
``` 

第二步：将以上查询结果当成临时表t(deptno,avgsal)与salgrade表进行表连接：t.avgsal between s.losal and s.hisal; 
``` 
select // 我们先得出部门编号、部门名称和薪水等级，因此要把表t和表salgrade联立 
t.deptno,t.dname,s.grade 
from // 第一步得到的各部门平均工资表与表salgrade联立起来 
(select 
e.deptno,d.dname,avg(e.sal) as avgsal 
from 
emp e 
join 
dept d 
on 
e.deptno = d.deptno 
group by 
e.deptno,d.dname) t 
join 
salgrade s 
on // 这里注意了，使用了between指令，t表中的平均工资和s表中的最低、最高工资如何匹配呢？ 
t.avgsal between s.losal and s.hisal; // 当t表中某个部门的平均工资a介于该部门的最低工资a1与最高工资a2之间（a1 < a < a2），则s表中的工资等级grade与该记录连接起来。 
``` 
第三步：将以上查询结果当成一张临时表t，找出最低的工资等级 
``` 
select // 得到平均薪水的最低等级 
min(t.grade) as minGrade 
from (select 
t.deptno,t.dname,s.grade 
from 
(select 
e.deptno,d.dname,avg(e.sal) as avgsal 
from 
emp e 
join 
dept d 
on 
e.deptno = d.deptno 
group by 
e.deptno,d.dname) t 
join 
salgrade s 
on 
t.avgsal between s.losal and s.hisal) t; 
``` 

第四步：当我们知道最低的工资等级是3，那么我们就可以求出该工资等级下的部门编号和部门名称 
``` 
select // 我们最终要得到平均薪水等级最低的部门编号与名称，及其工资等级 
t.deptno,t.dname,s.grade 
from 
(select // 先得出部门编号、名称与对应的的平均工资 
e.deptno,d.dname,avg(e.sal) as avgsal 
from 
emp e 
join 
dept d 
on 
e.deptno = d.deptno 
group by 
e.deptno,d.dname) as t 
join // 再使用between指令与表salgrade的工资等级联立 
salgrade s 
on 
t.avgsal between s.losal and s.hisal 
where // where指令过滤出工资等级=3的部门 
s.grade = 3; 
``` 

## 8. 取得比普通员工（员工代码没有在mgr上出现的）的最高薪水还要高的经理人姓名 

第一步：找出经理人（员工代码没有在mgr上出现的） 
1.1 先找出mgr有哪些人,注意了，最终结果有null值！ 
``` 
select distinct mgr from emp; // distinct（中文就是不同的、差异的）指令会去掉重复的行 
``` 

1.2 现在找出普通员工（员工代码没有在mgr上出现的） 
注意：not in的结果集（也就是1.1的结果）中出现null，则查询结果为null 
``` 
select * 
from 
emp 
where 
empno not in(select distinct mgr from emp); //not in 不会自动忽略空值 

select * 
from 
emp 
where 
empno in(select distinct mgr from emp); //in 会自动忽略空值 
``` 

正确做法： 
``` 
//手动忽略空值 
select * 
from 
emp 
where 
empno not in(select distinct mgr from emp where mgr is not null); 
``` 
1.3 从普通员工里面找出最高的薪水数额 
``` 
select 
max(sal) as maxsal 
from 
emp 
where 
empno not in(select distinct mgr from emp where mgr is not null); 
``` 

1.4 选出工资比普通员工最高薪水数额的经纪人的名字 
``` 
select 
ename 
from 
emp 
where \\ 过滤条件 
sal > (select 
max(sal) as maxsal 
from 
emp 
where 
empno not in(select distinct mgr from emp where mgr is not null)); 
``` 

## 9. 取得薪水最高的前五名员工 
``` 
select 
* 
from 
emp 
order by 
sal desc \\之前说过的，desc表示降序排序 
limit 0,5; \\这个之前也说过的，limit表示返回指定的记录数，从第一行开始取前五行 
``` 

## 10. 取得薪水最高的第六到第十的员工 
``` 
select 
* 
from 
emp 
order by 
sal desc 
limit 5,5; 
``` 

## 11. 取得最后(最新)入职的5名员工 
``` 
select 
* 
from 
emp 
order by 
hiredate desc 
limit 5; 
``` 

## 12. 取得每个薪水等级有多少员工 
### 步骤： 
第一步：查询每个员工的薪水等级 
``` 
select \\ 首先我们先得到每个员工对应的工资等级，因此先得到包含ename和grade的表 
e.ename,s.grade 
from 
emp as e 
join 
salgrade as s 
on 
e.sal between s.losal and s.hisal 
order by \\按照薪水等级升序排序 
s.grade; 
``` 

第二步：将以上结果当成临时表t(ename,grade) 
``` 
select \\ 最后我们要得到薪水等级以及每一个薪水等级下的员工数量 
t.grade,count(t.ename) as tot_emp \\ 因此我们使用count指令对每一个薪水等级下的员工数量进行计数 
from 
(select 
e.ename,s.grade 
from 
emp as e 
join 
salgrade as s 
on 
e.sal between s.losal and s.hisal 
order by 
s.grade) as t \\ 将以上结果当成临时表t(ename,grade) 
group by \\ 因此我们以薪水等级作为分组 
t.grade; 
``` 
## 14. 列出所有员工及领导的名字 
``` 
select 
e.ename,b.ename as leadername 
from 
emp e 
left join \\ left join，左连接。e作为左表，因此b作为右表 
emp b 
on 
e.mgr = b.empno； 
``` 
