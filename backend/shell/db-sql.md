## mysql命令
> 键入命令mysql -u root -p， 回车后提示你输入密码，然后回车即可进入到mysql中了
```sql
# 注意，如果是连接到另外的机器上，则需要加入一个参数-h机器IP
mysql （-h）-u 用户名 -p 用户密码

# 连接本机13306端口数据库
mysql -h 127.0.0.1 -P 13306 -u root -p

# 选择数据库
use 数据库名

# 查看数据库版本
select version();

# 显示数据库列表
# 命令后一般要加;表示命令结束
show databases;

# 查看数据表
show tables;

# 查看表结构(字段)
desc 表名;

# 删除数据表
drop table if exists 表名;


# 数据表数据操作
# 添加数据
insert into students(name) values('张三');

# 修改数据
# 修改`id为5`的学生数据，姓名改为 狄仁杰，年龄改为20
update students set name='狄仁杰', age=20 where id=5;

# 清空登录时间字段
update sys_user set login_time=null where user_id = 100

# 删除id为6的学生数据
delete from students where id=6;

# 删除学生表
drop table students;


# 查询学生的姓名、性别、年龄
# 条件查询, 分页查询: limit start, count
# (start从0开始)
# 从第一条记录开始查询, 并返回20条数据
select name, sex, age from students where name='小乔' and age=20 limit 0, 20;

# in 查询
select * from students where hometown in('北京', '上海', '深圳');

# between and查询(包括18和25岁学生)
select * from students where age between 18 and 25;

# 找出user_id在100-200最大的用户(返回1个)
select * from t_user where user_id BETWEEN 100 AND 200 order by user_id desc limit 0, 1;

# sql即可按单个字段排序,也可多个字段排序,多字段排序,优先级高的放前面,优先级低的放后面.
# SELECT * FROM t_user WHERE del_flag=0 AND (login_name = ? AND user_id > ?) GROUP BY sex ORDER BY user_type ASC,status ASC,create_time DESC
select * from t_user where create_time>'2023-01-10' order by user_type asc,user_id desc limit 100

# 查询所有商品信息,包含商品分类
select * from goods go left join category ca on go.typeId=ca.typeId;

# 扩充需求: 以分类为主展示所有内容(以哪张表为主表, 显示结果上是有区别的!)
select * from category ca left join goods go on ca.typeId=go.typeId; 

# json字段查询, json字段为config_josn, json值假设为: {"province":"广东省","city":"广州市"}
select * from sys_user where json_extract(config_json, '$.province') = "广东省"
```

## 建表sql
```sql
drop table if exists sys_user;
CREATE TABLE `sys_user` (
    -- 自增主键
    `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'id',
    `dept_id` INT NULL COMMENT '部门ID',

    -- int, long类
    `type` TINYINT NOT NULL DEFAULT '0' COMMENT '类型（1.平台用户; 2.app用户; ）',
    `status` TINYINT NOT NULL DEFAULT '0' COMMENT '状态（0停用 1启用）',

    -- 账号, 密码
    `user_name` varchar(50) not null comment '用户账号',
    `password` varchar(100) not null default '' comment '密码',


    -- 业务字段
    `user_no` VARCHAR(50) NULL COMMENT '用户编号' COLLATE 'utf8mb4_unicode_ci',
    `avatar` varchar(200) null comment '头像地址',

    -- 时间类
    `log_date` DATE NOT NULL COMMENT '日期',
    `log_time` TIME NOT NULL COMMENT '时间',
    `last_login_time` DATETIME NULL COMMENT '最后登陆时间',
    `price` DECIMAL(8,2) NULL COMMENT '钱包余额',

    -- 通用字段
    `config_json` json null comment '配置json',
    `remark` TEXT NULL COMMENT '备注' COLLATE 'utf8mb4_unicode_ci',
    `deleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '逻辑删除',
    `tenant_id` BIGINT NOT NULL DEFAULT '0' COMMENT '租户编号',
    `creator` VARCHAR(64) NULL DEFAULT NULL COMMENT '创建者' COLLATE 'utf8mb4_unicode_ci',
    `create_time` DATETIME NOT NULL COMMENT '添加时间',
    `update_time` DATETIME NULL DEFAULT NULL COMMENT '更新时间',
    `updater` VARCHAR(64) NULL DEFAULT NULL COMMENT '更新者' COLLATE 'utf8mb4_unicode_ci',
    PRIMARY KEY (`id`) USING BTREE,
    INDEX `user_name` (`user_name`) USING BTREE
)
COMMENT='用户表'
COLLATE='utf8mb4_unicode_ci'
ENGINE=InnoDB
AUTO_INCREMENT=1
;
```


## redis命令
```bash
# 查看redis版本
redis-server -v
redis-cli --version

#  redis常用命令
#  客户端连接, 连接本机
redis-cli

# 连接指定服务器, 指定端口, 指定密码-a
redis-cli -h 127.0.0.1 -p 6379 -a your_password

# 参看各种信息
info

# 退出
exit

# Redis默认是有16个数据库的（0~15）通过select命令来切换数据库
# 选择数据库(第三个)
select 2

# 清除屏幕输出
clear

# 查找所有keys
keys *

# 往数据库设置string类型值
set 'key_name' 'test value'

#  查找包括wx_的所有键
keys *wx_*

# 获取键key的值
get 'key'

# 获取键的ttl
ttl key

# 删除key, 一次只能删除一个
del key


# list集合类命令
# 获取到集合里元素的总个数
llen 'test:zoning:tree:all'              
```

---------------------
- [MySQL数据库常用命令总结](https://zhuanlan.zhihu.com/p/476887245)
- [redis 命令参考](http://redisdoc.com/)
- [mysql中json的使用方式详解](https://www.jb51.net/article/282049.htm#_label0)
- [MySQL中JSON_ARRAYAGG和JSON_OBJECT函数功能和用法](https://www.jb51.net/database/29843897g.htm)
