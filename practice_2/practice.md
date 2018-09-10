## 数据查看
查看学生表s：
```
select
    *
from
    s;
```
查看课程表c：
```
select
    *
from
    c;
```
查看学生选课表sc：
```
select
    *
from
    sc;
```

## 13.1 找出没选过“黎明”老师的所有学生姓名
1.先找出黎明老师授课的编号

```
select 
    cno
from
    c
where
    c.cteacher = "黎明";
```
2.从表sc找出学过黎明老师的课的学生学号

```
select
    sno
from
    sc
where
    cno = (select 
                cno
            from
                c
            where
                c.cteacher = "黎明");
```
3.最后就可以找出没有学过黎明老师的课的学生名字
```
select
    *
from                   \\
    s
where sno not in
    (select
        sno
    from
        sc
    where
        cno = (select 
                    cno
                from
                    c
                where
                    c.cteacher = "黎明"));
```
### 总结：
我们一定要搞清楚每个表之间的关系，弄清每个表的主键和外键（一般都是ID）。

这道题的思路是这样的：先从表c找出黎明老师授课的编号 → 表sc作为表c与表s的连接，所以从表sc找出学过黎明老师的课的学生学号 → 最后从学生表s找出学过黎明老师的课的学生名字（就算是找出“没学过”的也一样的思路）
    
## 13.2 列出2门以上（含两门）不及格学生姓名及平均成绩   
### 步骤：
1.先列出2门以上（含两门）不及格学生姓名
```
//t1
select                                   \\先从表sc选出学生学号，从表s选出学生姓名，并对学生学号出现的次数进行计数，以此作为分组
    sc.sno,s.sname,count(*) as classNum
from 
    sc
join 
    s
on                          \\ 学生表s是没有成绩的，所以要把表s和表sc连接起来
    sc.sno = s.sno
where
    sc.scgrade < 60
group by 
    sc.sno
having                       \\过滤出2门以上（含两门）不及格的学生
    classNum >= 2;
```
2.对每位学生进行平均成绩的计算
```
select
    sc.sno,avg(sc.scgrade) as avgscgrade
from
    sc
group by
    sc.sno;
```    

3.现在已经知道2门以上（含两门）不及格学生姓名(临时表t1)与每位学生的平均成绩(临时表t2)，就可以知道2门以上（含两门）不及格学生姓名及其平均成绩
```
select 
    t1.sname,t2.avgscgrade
from
    (select                                   
        sc.sno,s.sname,count(*) as classNum
    from 
        sc
    join 
        s
    on                          
        sc.sno = s.sno
    where
        sc.scgrade < 60
    group by 
        sc.sno
    having                       
        classNum >= 2) as t1
join
    (select
        sc.sno,avg(sc.scgrade) as avgscgrade
    from
        sc
    group by
        sc.sno) as t2
on
    t1.sno = t2.sno;
```

## 13.3 既学过1号课程又学过2号课程所有学生的姓名

### 步骤
1.先选出学过1号课程的学生学号
```
select 
    sno 
from 
    sc 
where 
    cno = 1;
```
2.再选出学过2号课程的学生学号
```
select 
    sno 
from 
    sc 
where 
    cno = 2;
```
3.最后就可以选出既学过1号课程又学过2号课程所有学生的姓名
```
select 
    *
from 
    sc 
join
    s
on
    sc.sno = s.sno
where  \\ 首先同一条记录是不可能既出现cno = 1又出现cno = 2的，那么怎么样才能表示既学过1号课程又学过2号课程的学生？于是选出同时学过1号课程（cno=1）与学过2号课程的学生学号（sc.sno in (select sno from sc where cno = 2))
    cno = 1                                       \\ cno只存在在表sc，因此不用加前缀
and 
    sc.sno in (select sno from sc where cno = 2); \\ 但是sno在表sc和表s都存在，因此要加前缀
    
