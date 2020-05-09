# mysql之常用语句

## DQL(数据查询语句)

* select字句的顺序
```
select ->from ->where ->group by ->having ->order by ->limit
```

* 模糊查询 like
    * %：任意字符出现任意次数
    * _:只匹配单个字符

* 汇总 

* 分组:group by

* 分页:limit

* 排序:order by

* 子查询

* 连接查询：join

* 组合查询：union

## DML(数据操纵语句)
* insert：insert into table_name ( field1, field2,...fieldN ) VALUES ( value1, value2,...valueN );
* delete:delete from table_name [WHERE Clause]
* update:update table_name set userName='kobe',age=42 where id=1


## DDL(数据定义语句)
* 增加字段：add (通过after指定位置)
>alter table table_name add field_name field_type after field_name
* 删除字段：drop
>alter table table_name  DROP field
* 修改字段类型：mofidy 
>alter  table table_name modify [COLUMN] 字段名 新数据类型 新类型长度  新默认值  新注释;
>modify可以用来修改字段类型，是否必填，注释。例如：当修改比否非必填时，字段类型和注释如果不修改，也需要带上当前值，否则会被清空
* 修改字段名称 change
>alter  table table1 change old_column new_column varchar(100) DEFAULT 1.2 COMMENT '注释';
* 修改字段默认值
> alter table talble_name alter field set DEFAULT 1000;


## DCL(数据控制语句):管理用户，授权
### 1.管理用户
* 添加用户：
语法：CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';
* 删除用户：
语法：DROP USER '用户名'@'主机名';
* 修改用户密码：
    UPDATE USER SET PASSWORD = PASSWORD('新密码') WHERE USER = '用户名';
    UPDATE USER SET PASSWORD = PASSWORD('abc') WHERE USER = 'lisi';
    SET PASSWORD FOR '用户名'@'主机名' = PASSWORD('新密码');
    SET PASSWORD FOR 'root'@'localhost' = PASSWORD('123');


### 2.权限管理
* 授予权限
grant 权限列表 on 数据库名.表名 to '用户名'@'主机名';
-- 给张三用户授予所有权限，在任意数据库任意表上
GRANT ALL ON . TO 'zhangsan'@'localhost';

### 3.撤销权限
-- 撤销权限：
revoke 权限列表 on 数据库名.表名 from '用户名'@'主机名';
REVOKE UPDATE ON db3.account FROM 'lisi'@'%';
 

## Q&A
* delete和truncate比较
    * delete：删除表中的行，不删除表本身。
    * truncate：如果想删除表中所有行，truncate更快。truncate实际上是删除原有的表并重新创建一张表，而不是逐行的删除数据。

