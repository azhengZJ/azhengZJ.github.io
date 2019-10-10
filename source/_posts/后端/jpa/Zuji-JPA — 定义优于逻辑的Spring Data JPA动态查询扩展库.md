---
title: Zuji-JPA — 定义优于逻辑的Spring Data JPA动态查询扩展库
date: 2019-10-08 14:10:00
author: azheng
img: http://static.blinkfox.com/hexoblog_20180920_proxy.jpg
cover: true
top: true
categories: 后端
tags:
  - Java
  - JPA
  - Zuji
---

- [使用文档: https://azhengZJ.github.io/zuji-jpa](https://azhengZJ.github.io/zuji-jpa)

QQ交流群:`758629787`

比mybatis更好用的超轻量的spring-data-jpa扩展库。

全部基于静态工具类方法实现，程序启动无需加载任何Class.

单层级的动态条件查询只需定义入参实体类，不用写具体java实现代码，也不用写sql即可信手拈来。

多层级嵌套复杂的动态条件查询使用超简洁的链式编程和语义化编程即可轻松实现。

使用zuji-jpa可以大大简化开发，节省更多的时间让你专注于业务。


## 示例  

Zuji-JPA查询全部基于Specification进行开发，所以使用之前必须要继承JpaSpecificationExecutor接口
```java
public interface UserRepository extends JpaSpecificationExecutor<User> {

}
```

### 多层嵌套复杂条件查询

此查询类似于mybatis-plus的条件构造器。
以下示例包含：动态条件查询 + or 嵌套条件+ 排序+ 分页

```java
public Page<User> list(ReqUserListDTO params) {
   Specification<User> spec = SpecificationUtils.where(e -> {
       if (!params.getUserType().equals(UserType.ALL)) {
           e.eq("userType", params.getUserType());
       }
       if (params.getPriority() > 0) {
           e.eq("priority", params.getPriority());
       }
       if (params.getRange() != null) {
           e.eq("assigneeId", AuthHelper.currentUserId());
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
public Page<User> list(ReqUserListDTO params) {
   Specification<User> spec = SpecificationUtils.where(e -> {
       e.eq("userType", params.getUserType())
        .eq("priority", params.getPriority())
        .eq("assigneeId", AuthHelper.currentUserId())
        .or(e2 -> e2.eq("status", "1").eq("status", "2"))
        .eq("deleted", 0);
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
	user
WHERE
	userType = 'ADMIN' 
	AND priority = 1 
	AND assigneeId = 11 
	AND ( STATUS = 1 OR STATUS = 2 ) 
	AND deleted = 0 
ORDER BY
	create_time DESC 
	LIMIT 0,10
```

### 单层多条件查询

未完待续。。。敬请期待


基于spring data jap Specification 查询封装，封装精简 灵活 ，扩展性非常强。
需要源码的  请加qq交流群（758629787） 免费获取源码。


## 开源许可证

本 `Zuji-JPA` 的 Spring Data JPA 扩展库遵守 [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0) 许可证。
