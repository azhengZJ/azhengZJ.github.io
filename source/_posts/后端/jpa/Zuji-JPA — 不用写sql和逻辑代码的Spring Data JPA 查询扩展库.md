---
title: Zuji-JPA — 不用写sql和逻辑代码的 Spring Data JPA 复杂查询扩展库
date: 2019-10-09 14:10:00
author: azheng
img: http://static.blinkfox.com/blog/2019/08/20jpa-20190820.png
cover: true
categories: 后端
tags:
  - Java
  - JPA
  - Zuji
---

比mybatis更好用的超轻量的spring-data-jpa扩展库。

全部基于静态工具类方法实现，程序启动无需加载任何bean.

以后再也不用写sql，单层级的动态条件查询不用写具体实现的java代码即可信手拈来。

多层级嵌套复杂的动态条件查询使用超简洁的链式编程和语义化编程即可轻松实现。

- [使用文档: https://azhengZJ.github.io/fenix](https://azhengZJ.github.io/zuji-jpa)

## 示例  

类似于mybatis-plus的条件构造器
动态条件查询 + or 嵌套条件+ 排序+ 分页

### 多层嵌套复杂条件查询

```java
public Page<Users> list(ReqUserListDTO params) {
   Specification<Users> spec = SpecificationUtils.where(e -> {
       if (!params.getUserType().name().equals(UserTypeEnum.ALL.name())) {
           e.eq("userType", params.getUserType().name());
       }
       if (params.getPriority() > 0) {
           e.eq("priority", params.getPriority());
       }
       if (params.getRange() != null) {
           e.eq("assigneeId", authService.currentManagerId());
       }
       e.or(e2 -> {
           e2.eq("status", "1");
           e2.eq("status", "2");
       });
       e.eq("deleted", 0);
   });
   Sort sort = Sort.by("createTime").descending();
   return repository.findAll(spec, params.pageRequest(sort));
}
```
如果没有条件判断也可以写成这样，链式编程

```java
public Page<Users> list(ReqUserListDTO params) {
   Specification<Users> spec = SpecificationUtils.where(e -> {
       if (!params.getUserType().name().equals(UserTypeEnum.ALL.name())) {
           e.eq("userType", params.getUserType().name());
       }
       if (params.getPriority() > 0) {
           e.eq("priority", params.getPriority());
       }
       if (params.getRange() != null) {
           e.eq("assigneeId", authService.currentManagerId());
       }
       e.or(e2 -> e2.eq("status", "1").eq("status", "2")).eq("deleted", 0);
   });
   Sort sort = Sort.by("createTime").descending();
   return repository.findAll(spec, params.pageRequest(sort));
}
```

等同于sql

```sql
SELECT
	* 
FROM
	users
WHERE
	userType = 'admin' 
	AND priority = 1 
	AND assigneeId = 11 
	AND ( STATUS = 1 OR STATUS = 2 ) 
	AND deleted = 0 
ORDER BY
	create_time DESC 
	LIMIT 0,10
```

### 单层多条件查询

未完待续。。。


基于spring data jap Specification 查询封装，封装精简 灵活 ，扩展性非常强。
需要源码的  请加qq交流群（758629787） 免费获取源码。



## 开源许可证

本 `Zuji-JPA` 的 Spring Data JPA 扩展库遵守 [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0) 许可证。
