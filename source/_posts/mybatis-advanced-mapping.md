---
title: MyBatis框架基于Annotation注解的一对多关联映射
date: 2018-08-19 03:44:43
tags: 
	- mybatis
	- oneToMany
---



# 数据结构

entity有Semester和Schedule 其中多个Schedule映射一个Semester 即一个Semester指向多个Schedule

Semester:

```java
@Table(name = "`cms_semester`")
public class Semester {
    @Id
    @Column(name = "`id`")
    @GeneratedValue(generator = "JDBC")
    private Integer id;

    @Column(name = "`end_date`")
    private Date semesterStart;

    @Column(name = "`start_date`")
    private Date semesterEnd;

    @Column(name = "`title`")
    private String title;

    private List<Schedule> schedule;
    
    //省去getter与setter
 }
```

Schedule：

```java
@Table(name = "`cms_schedule`")
public class Schedule {
    @Id
    @Column(name = "`id`")
    @GeneratedValue(generator = "JDBC")
    private Integer id;

    @Column(name = "`target`")
    private String target;

    @JsonIgnore
    @Column(name = "`semester_id`")
    private Integer semesterId;
    //省去getter与setter
 }
```

# 使用@many 进行多表查询#

先需要按照时间取出数据库中Semester数据，并将其指向Schedule数据一并取出，存于schedule列表中。

MyBatis提供了@Many注解来加载一对多的注解，运用Nested-SELECT声明。

SemesterDAO:

```java
public interface SemesterDAO extends tk.mybatis.mapper.common.Mapper<Semester> {
    @Results({
            @Result(column="id", property="id", jdbcType= JdbcType.INTEGER, id=true),
            @Result(column="end_date", property="semesterEnd", jdbcType=JdbcType.DATE),
            @Result(column="start_date", property="semesterStart", jdbcType=JdbcType.DATE),
            @Result(column="title", property="title", jdbcType=JdbcType.VARCHAR),
            @Result(column="id", property="schedule",many = @Many(select = "isdc.isdcssm.dao.ScheduleDAO.selectBySemesterId")),
    })
//该
    @Select("select * from cms_semester where start_date < #{date} and end_date > #{date}")
    Semester selectByDate(Date date);
}
```



ScheduleDAO：

```java
public interface ScheduleDAO extends tk.mybatis.mapper.common.Mapper<Schedule> {

    @Results({
            @Result(column="id", property="id", jdbcType= JdbcType.INTEGER, id=true),
            @Result(column="target", property="target", jdbcType=JdbcType.VARCHAR),
            @Result(column="semester_id", property="semesterId", jdbcType=JdbcType.INTEGER),
    })
    @Select("select * from cms_schedule where semester_id = #{semesterId} ")
    List<Schedule> selectBySemesterId(int semesterId);
}
```

## 一对一映射@one

MyBatis提供@one的注解来加载一对一的注解，和运用Nested-Select声明

使用方法与@Many相同，返回结果为单一对象，不过多累述。