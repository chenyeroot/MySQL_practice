```
有三个表s（学生表）、c（课程表）、sc（学生选课表）
S（SNO,SNAME）代表 （学号，姓名）
C（CNO,CNAME,CTEACHER）代表（课号，课名，教师）
SC（SNO,CNO,SCGRADE）代表（学号，课号，成绩）

问题:
1.找出没选过“黎明”老师的所有学生姓名
2.列出2门以上（含两门）不及格学生姓名及平均成绩
3.既学过1号课程又学过2号课程所有学生的姓名
```
## 数据插入
```
create table s(
    sno int(4) primary key auto_increment,
    sname varchar(32)
);

insert into s(sname) values ('zhangsan');
insert into s(sname) values ('lisi');
insert into s(sname) values ('wangwu');
insert into s(sname) values ('zhaoliu');

create table c(
    cno int(4) primary key auto_increment,
    cname varchar(32),
    cteacher varchar(32)
);

insert into c(cname,cteacher) values ('Java','吴老师');
insert into c(cname,cteacher) values ('C++','王老师');
insert into c(cname,cteacher) values ('C##','张老师');
insert into c(cname,cteacher) values ('MySQL','郭老师');
insert into c(cname,cteacher) values ('Oracle','黎明');

create table sc(
    sno int(4),
    cno int(4),
    scgrade double(3,1),
    constraint sc_sno_cno_pk primary key(sno,cno),
    constraint sc_sno_fk foreign key(sno) references s(sno),
    constraint sc_cno_fk foreign key(cno) references c(cno)
);

insert into sc(sno,cno,scgrade) values (1,1,30);
insert into sc(sno,cno,scgrade) values (1,2,50);
insert into sc(sno,cno,scgrade) values (1,3,80);
insert into sc(sno,cno,scgrade) values (1,4,90);
insert into sc(sno,cno,scgrade) values (1,5,70);

insert into sc(sno,cno,scgrade) values (2,2,80);
insert into sc(sno,cno,scgrade) values (2,3,50);
insert into sc(sno,cno,scgrade) values (2,4,70);
insert into sc(sno,cno,scgrade) values (2,5,80);

insert into sc(sno,cno,scgrade) values (3,1,60);
insert into sc(sno,cno,scgrade) values (3,2,70);
insert into sc(sno,cno,scgrade) values (3,3,80);

```
