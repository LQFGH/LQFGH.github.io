---
layout:     post
title:      "MyBatis-传参处理"
subtitle:   "各种场景下对传参的处理"
date:       2019-01-22 23:10
author:     "LQFGH"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.3
catalog:    true
tags:
  - MyBatis
---


#### 场景一：

```java
Employee selectEmp(@Param("id") Integer id,String lastName);
```

```xml
id->#{id/param1}
lastName->#{param2}
```


#### 场景二：

```java
Employee selectEmp(@Param("id") Integer id,@Param("e") Employee emp);
```

```xml
id->#{param1}
emp.lastName->#{param1.lastName/e.lastName}
```

#### 场景三：

```java
Employee selectEmp(List ids);
```

```xml
<!-- list中的第一个值 -->
-> #{list[0]}
```

> !特别注意
> 
> 如果是Collection、List、Set类型或是数组，会把list或数组封装到map中
> List的key是list，数组的key是array,Set 是set

