---
title: JdbcTemplate和NamedParameterJdbcTemplate的区别
date: 2019-06-14 09:44:50
tags:
- Show All
- jdbc
header-img: "Demo.png"
---

## JdbcTemplate和NamedParameterJdbcTemplate的区别

1. JdbcTemplate预编译位置是用"?",NamedParameterJdbcTemplate预编译位置是用":属性名".
2. JdbcTemplate设置参数使用的是的Object数组，NamedParameterJdbcTemplate使用的是MapSqlParameterSource

#### NamedParameterJdbcTemplate

```java
@Resource
protected NamedParameterJdbcTemplate npJdbcTemplate;

String sql = baseSql +" and exam_id in (:asgmtIds) and exam_sponsor = :teacherId";
MapSqlParameterSource parameters = new MapSqlParameterSource();
List<Integer> collect =Arrays.stream(asgmtIds).map(Integer::valueOf)
.collect(Collectors.toList());
parameters.addValue("asgmtIds", collect);
parameters.addValue("teacherId", teacherId);
npJdbcTemplate.query(sql, parameters, new BeanPropertyRowMapper<(AsgmtsDO.class));
```

#### JdbcTemplate

```java
@Resource
protected JdbcTemplate jdbcTemplate;

String sql = baseSql + "paper_id=?";
Object obj = new Object[]{asgmtId};
jdbcTemplate.query(sql,obj,new BeanPropertyRowMapper<>(PaperDO.class));
```

