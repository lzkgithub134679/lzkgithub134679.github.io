---
title: map集合排序
date: 2019-06-12 11:43:39
tags:
- Show All
- 数据结构
header-img: "Demo.png"
---

# stream的map排序

**map根据key正序排序**

```java
map.entrySet().stream().sorted(Comparator.comparing(e->e.getKey()))
.forEach(System.out::println);
```



**map根据key倒序排序**

```java
map.entrySet().stream().sorted(Collections.reverseOrder
(Map.Entry.comparingByKey())).forEach(System.out::println);
```

**map根据value正序排序**

```java
map.entrySet().stream().sorted(Comparator.comparing(e->e.getValue()))
.forEach(System.out::println);
```

**map根据value倒序排序**

```java
map.entrySet().stream().sorted(Collections.reverseOrder
(Map.Entry.comparingByValue())).forEach(System.out::println);
```

