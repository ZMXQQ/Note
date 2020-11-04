学生<img src="C:\Users\李佳庆\Desktop\APP\笔记本\图片库\image-20201022185723584.png" alt="image-20201022185723584" style="zoom: 67%;" />老师![image-20201022185740839](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\image-20201022185740839.png)课程

![image-20201022185809186](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\image-20201022185809186.png)成绩

![image-20201022185825608](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\image-20201022185825608.png)

----------------------------------

*#1、查询"01"课程比"02"课程成绩高的学生的信息及课程分数*	
select st.*, s1.s_score from student st,score s1, score s2 where  s1.s_score > s2.s_score and s1.c_id = '01' and s2.c_id = '02' and s1.s_id = st.s_id and s2.s_id = st.s_id

> 内连接：不用join on，直接用where。



*#3、查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩*

select st.s_id,st.s_name,avg(sc.s_score)        from student st left join score sc on sc.s_id = st.s_id      group by st.s_id,st.s_name having avg(sc.s_score)  >= 60  

> Having：筛选分组之后的数据