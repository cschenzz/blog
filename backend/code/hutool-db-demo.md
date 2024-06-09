## pom.xml导入
```xml
<!-- https://hutool.cn/docs/#/ -->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.26</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.zaxxer/HikariCP -->
<!-- Java 11+ use 5.1.0, Java8 use 4.0.3 -->
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>5.1.0</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <optional>true</optional>
</dependency>
```

## 获取数据源
```java
import cn.hutool.db.Db;
import cn.hutool.db.Entity;
import cn.hutool.db.Page;
import cn.hutool.db.PageResult;
import cn.hutool.db.meta.MetaUtil;
import cn.hutool.db.meta.Table;
import cn.hutool.db.sql.Condition;
import cn.hutool.db.sql.Direction;
import cn.hutool.db.sql.Order;
import cn.hutool.db.sql.SqlBuilder;
import cn.hutool.db.ds.simple.SimpleDataSource;
import cn.hutool.json.JSONUtil;
import cn.hutool.log.level.Level;

import cn.hutool.core.text.StrBuilder;
import cn.hutool.core.util.StrUtil;
import cn.hutool.core.util.ArrayUtil;
import cn.hutool.core.date.DateField;
import cn.hutool.core.date.DateTime;
import cn.hutool.core.date.DateUtil;

import java.util.*;
import javax.sql.DataSource;
import java.sql.SQLException;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

/**
 * 获取数据源
 *
 * @return
 */
private DataSource getDs() {
    // DataSource ds = SpringUtils.getBean("masterDataSource");
    DataSource ds = new SimpleDataSource(
            "jdbc:mysql://localhost:3306/test-db?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai",
            "root",
            "root",
            "com.mysql.cj.jdbc.Driver"
    );
    // ---------------------------------------
    // 获得当前库的所有表的表名
    List<String> tableNames = MetaUtil.getTables(ds);
    log.info("{}", tableNames);

    // 获得表结构 表结构封装为一个表对象，里面有Column对象表示一列，列中有列名、类型、大小、是否允许为空等信息
    tableNames.forEach(oo -> {
        Table table = MetaUtil.getTableMeta(ds, oo);
        log.info("-1.---{}-----,\r\n{}", oo, JSONUtil.toJsonPrettyStr(table));
        log.info("----打印字段----");
        var columns = table.getColumns();
        columns.forEach(ss -> {
            // 表字段转化成驼峰命名, user_id-->userId
            log.info("{}-->{}", ss.getName(), StrUtil.toCamelCase(ss.getName()));
        });
    });
    // ---------------------------------------
    return ds;
}

private DataSource getHikariDs() {
    // 使用Hikari连接池
    HikariConfig config = new HikariConfig();
    config.setJdbcUrl("jdbc:mysql://127.0.0.1:3306/test-db?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8");
    config.setUsername("root");
    config.setPassword("root");
    config.setDriverClassName("com.mysql.cj.jdbc.Driver");
    // config.addDataSourceProperty("cachePrepStmts", "true");
    // config.addDataSourceProperty("prepStmtCacheSize", "250");
    // config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
    // HikariDataSource ds = new HikariDataSource(config);
    return new HikariDataSource(config);
}
```

## 查询数据
```java
// throws SQLException
Db db = Db.use(getDs());
// 设置全局是否在结果中忽略大小写
// cn.hutool.db.GlobalDbConfig.setCaseInsensitive(false);
// db.setCaseInsensitive(false);
// 设置全局配置, 通过debug日志显示SQL(格式化显示, 并打印参数)
cn.hutool.db.GlobalDbConfig.setShowSql(true, true, true, Level.DEBUG);

DateTime beginTime = DateUtil.beginOfDay(DateTime.of("2023-05-09", "yyyy-MM-dd"));
DateTime endTime = DateUtil.endOfMonth(beginTime);

// List<Entity> entityList = db.query("select * from sys_user where login_date>? order by user_id desc limit 10", DateTime.now().offsetNew(DateField.DAY_OF_MONTH, -7));
List<Entity> entityList = db.query("select s.* from sys_user s where s.user_id>? and s.login_date between ? and ? order by s.user_id desc", 0, beginTime, endTime);
List<Entity> ddList = entityList.stream().peek(t -> {
            Date loginDateTime = t.getDate("login_date");
            t.set("login_date_text", DateUtil.formatDateTime(loginDateTime));
            t.set("days", DateUtil.between(loginDateTime, DateTime.now(), DateUnit.DAY));
            t.set("update_time_text", DateUtil.formatDateTime(t.getDate("update_time")));
        }
).collect(Collectors.toList());
log.info("-----data({})------\r\n{}", ddList.size(), JSONUtil.toJsonPrettyStr(ddList));

List<Dict> dictList = entityList.stream().map(oo -> {
            Date loginDateTime = oo.getDate("login_date");

            Dict mmDict = Dict.create();

            // 转换成驼峰式字段名
            // oo.getFieldNames().forEach(tt -> {
            //     mmDict.set(StrUtil.toCamelCase(tt), oo.get(tt));
            // });

            mmDict.set("loginDate", DateUtil.formatDateTime(loginDateTime));
            mmDict.set("userId", oo.getLong("user_id"));
            mmDict.set("nickName", oo.getStr("nick_name"));

            return mmDict;
        }
).toList();

// ===================================
// 过滤获取单条记录
Optional<Entity> optionalEntity = entityList.stream().filter(oo -> oo.getLong("user_id") == 1L).findAny();
if (optionalEntity.isPresent()) {
    Entity adminUser = optionalEntity.get();
    log.info("-----user------\r\n{}", adminUser);
} else {
    log.info("-----不存在------");
}

// 查询单条数据(sql查询有多条记录时只返回第一条, 没有数据返回null)
Entity entityOne = db.queryOne("select * from sys_user where user_id=?", 69);
// 打印字段
entityOne.keySet().forEach(xx -> log.debug(xx));

// 模糊查询
List<Entity> resultList = db.query("select * from sys_user where nick_name like ?", "%悟空%");

// 分页, 页码，0表示第一页, pageSize – 每页结果数
// 查询昵称为王姓开头的用户, 返回第一页, 每页20条记录
PageResult<Entity> result = db.page("select * from sys_user where nick_name like ?", new Page(0, 20), "王%");
```

## mysql中json字段的查询
```java
// https://blog.csdn.net/minshiwang/article/details/130769571
// select * from sys_user where json_extract(address, '$.province') = "河北省";
Db db = Db.use(getDs());
// 搜索tags包含'家'的用户
List<Entity> entityList = db.query("select * from sys_user where JSON_CONTAINS(address,JSON_OBJECT('tags', ?))", "家");
entityList.forEach(oo -> {
    oo.set("address", JSONUtil.parse(oo.getStr("address")));
});
Console.log(JSONUtil.toJsonPrettyStr(entityList));

// 查询厦门市公司, json字段map_result数据见下面地图数据解析
List<Entity> entityList = db.query("select * from cc_company where map_result-> '$.result.ad_info.city' = ?", "厦门市");
// List<Entity> entityList = db.query("select * from cc_company where json_extract(map_result,'$.result.ad_info.city') = ?", "厦门市");
```

## 计算表记录条数
```java
Db db = Db.use(getDs());
List<Entity> rowsList = db.query("select count(1) as r from oper_log where status=?", 1);
int rowsCount = rowsList.isEmpty() ? 0 : rowsList.get(0).getInt("r");
// 不能使用db.query来返回记录条数, 会报错: java.sql.SQLException: Can not issue executeUpdate() or executeLargeUpdate() with statements that produce result sets

// 使用count方法计算数据条数
long rows = db.count("select * from oper_log where status=?", 1);

// 使用queryOne实现(计算2022-12月份之后日志记录条数)
Entity entityRows = db.queryOne("select count(1) as r from oper_log where create_time>?", DateUtil.beginOfMonth(DateTime.of("2022-12", "yyyy-MM")));
int xRowsCount = entityRows.getInt("r");

// queryString(查询单条单个字段记录,并将其转换为String)
// count(0), count(1), count(*)都可以
// 使用queryNumber计算数量
Number xCount = db.queryNumber("select count(1) as r from sys_user where create_time>?", DateTime.of("2022-12-12", "yyyy-MM-dd"));
```


## 其他操作-crud
```java
// Db db = Db.use(getDs());
// 插入数据并返回自增主键
long id = db.insertForGeneratedKey(Entity.create("sys_user").set("name", "unitTestUser").set("age", 66));
long pid = db.executeForGeneratedKey("insert into sys_user(name, age) values (?, ?)", "铠", 21);
// 使用sql实现增删改
// 新增
int rows = db.execute("insert into sys_user values (?, ?, ?)", "张三", 59, 1);
// 删除
int rows = db.execute("delete from sys_user where name = ?", "张三");
// 更新
int rows = db.execute("update sys_user set age = ? where name = ?", 3, "张三");


// IN查询, 我们在执行类似于select * from sys_user where id in 1,2,3这类SQL的时候，Hutool封装如下
List<Entity> results = db.findAll(Entity.create("sys_user").set("id", new long[]{1, 2, 3}));

// 分页in查询, 指定表, 分页条件
// 查询第一页, 返回20条
PageResult<Entity> result = db.page(Entity.create("sys_user").set("name", CollUtil.newArrayList("czz", "陈某")), new cn.hutool.db.Page(0, 20));
log.debug("记录总数: {}, 页码: {}/{}, 每页显示: {}, result={}", result.getTotal(), result.getPage() + 1, result.getTotalPage(), result.getPageSize(), JSONUtil.toJsonPrettyStr(result));
```

## 事务
```java
Db.use(getDs()).tx(txDb -> {
    // 以下增删改都是针对表sys_user
    // 使用sql插入数据
    int rows = txDb.execute("insert into sys_user(name, age) values (?, ?)", "公孙离", 18);
    // 新增(表和字段)
    txDb.insert(Entity.create("sys_user").set("name", "unitTestUser2"));

    // update(更新的字段, 表和where条件)
    int rows = txDb.update(Entity.create().set("age", 79), Entity.create("sys_user").set("name", "unitTestUser2"));

    // 删除
    int delRows = txDb.del(
        Entity.create("sys_user").set("name", "unitTestUser") // 表+where条件
    );
});
```


## 多条件查询构造生成示例
```java
Db db = Db.use(getDs());

// and多条件
var whereList = List.of(
        "JSON_CONTAINS(config_json,JSON_OBJECT('tags', ?))",
        "type=?",
        "title=?"
);

// 构造sql
var sqlBuilder = StrBuilder.create("select * from tb_article");
if (!whereList.isEmpty()) {
    sqlBuilder.append(" where ")
            .append(String.join(" and ", whereList));
}

// 排序
sqlBuilder.append(" order by create_time desc");
cn.hutool.core.lang.Console.log(sqlBuilder);

PageResult<Entity> result = db.page(sqlBuilder.toString(), new Page(0, 20), "cpu", 9, "标题");
log.debug("记录总数: {}, 页码: {}/{}, 每页显示: {}, result={}", result.getTotal(), result.getPage() + 1, result.getTotalPage(), result.getPageSize(), JSONUtil.toJsonPrettyStr(result));
```


## SqlBuilder生成查询示例
```java
Db db = Db.use(getDs());

// DateTime beginTime = DateUtil.beginOfDay(DateUtil.parseDate("2023-07-01"));
// DateTime endTime = DateUtil.endOfDay(DateUtil.parseDate("2023-07-12"));

// and多条件构建
List<Condition> conditions = CollUtil.newArrayList(
        new Condition("user_type", "=", "0"),
        // new Condition("email", "qq.com", Condition.LikeType.Contains),
        new Condition("name", "in", "czz,tim,猪头强")
        // -------------between ? and ?可以使用>=? and <=?代替实现-------------
        // new Condition("create_time", ">=", beginTime),
        // new Condition("create_time", "<=", endTime)
);

SqlBuilder sqlBuilder = SqlBuilder.create()
        .select()
        .from("sys_user")
        // .where(ArrayUtil.toArray(conditions, Condition.class))
        // Condition... conditions可变参数需要传数组
        // 数组引用, 语法格式为: Type[]::new
        .where(conditions.toArray(Condition[]::new))
        .orderBy(new Order("create_time", Direction.DESC));
// 生成sql
String sqlText = sqlBuilder.build();
Object[] paramArray = sqlBuilder.getParamValueArray();
// 1. 查询
List<Entity> entityList = db.query(sqlText, paramArray);

// =============================
log.info("{}\r\n{}", sqlText, JSONUtil.toJsonPrettyStr(paramArray));
log.info("-----data({})------\r\n{}", entityList.size(), JSONUtil.toJsonPrettyStr(entityList));
// SELECT * FROM sys_user WHERE user_type = ? AND name in (?,?,?) ORDER BY create_time DESC
// ["0","czz","tim","猪头强"]

// =============================
// 2. 分页
PageResult<Entity> pageResult = db.page(sqlText, new Page(0, 20), paramArray);
log.debug("记录总数: {}, 页码: {}/{}, 每页显示: {}, result={}", pageResult.getTotal(), pageResult.getPage() + 1, pageResult.getTotalPage(), pageResult.getPageSize(), JSONUtil.toJsonPrettyStr(pageResult));

// =============================
// 3. Condition多条件查询
List<Entity> rowList = db.findBy("sys_user",
        new Condition("name", "in", "admin,chenzz,tim"),
        new Condition("user_type", "=", "0"),
        new Condition("email", "qq.com", Condition.LikeType.Contains)
        // new Condition("create_time", ">=", beginTime),
        // new Condition("create_time", "<=", endTime)
);
// =============================
```

## 在表中找出字段重复数据
```java
// 在数据库表中找出指定字段中有重复数据的记录
// 重复数据处理, 查找openid字段重复的记录
// select * from sys_user where wx_open_id in (select wx_open_id from sys_user group by wx_open_id having count(wx_open_id) > 1)
String sql = StrUtil.format("select * from {table} where {field} in (select {field} from {table} group by {field} having count({field}) > 1) order by {field} asc", Dict.create().set("table", "sys_user").set("field", "wx_open_id"));
Console.log(sql);
// List<Entity> entityList = db.query(sql);
```


## json字段数据示例
```json
// select * from tb_article where JSON_CONTAINS(config_json,JSON_OBJECT('tags', '国产'))
// select * from tb_article where json_extract(config_json,'$.type') = 99
{
    "type": 99,
    "category": "电子芯片",
    "tags": [
        "CPU",
        "国产",
        "龙芯",
        "MIPS"
    ]
}
```

## 腾讯地图根据经纬度逆地址解析接口返回的json数据
```json
// select * from tb_address where map_result-> '$.result.ad_info.city' = '厦门市'
{
    "result": {
        "ad_info": {
            "city": "厦门市",
            "name": "中国,福建省,厦门市,集美区",
            "adcode": "350211",
            "nation": "中国",
            "district": "集美区",
            "location": {
                "lat": 24.575976,
                "lng": 118.097407
            },
            "province": "福建省",
            "city_code": "156350200",
            "nation_code": "156",
            "phone_area_code": "0592"
        },
        "address": "福建省厦门市集美区诚毅北大街",
        "location": {
            "lat": 24.615434,
            "lng": 118.045612
        },
        "address_component": {
            "city": "厦门市",
            "nation": "中国",
            "street": "诚毅北大街",
            "district": "集美区",
            "province": "福建省",
            "street_number": ""
        },
        "address_reference": {
            "town": {
                "id": "350211103",
                "title": "后溪镇",
                "location": {
                    "lat": 24.629668,
                    "lng": 118.041328
                },
                "_dir_desc": "内",
                "_distance": 0
            },
            "street": {
                "id": "15342814244316509511",
                "title": "诚毅北大街",
                "location": {
                    "lat": 24.612911,
                    "lng": 118.046669
                },
                "_dir_desc": "东",
                "_distance": 68
            },
            "landmark_l2": {
                "id": "3233502942279593447",
                "title": "金砖未来创新园",
                "location": {
                    "lat": 24.615199,
                    "lng": 118.045697
                },
                "_dir_desc": "内",
                "_distance": 0
            },
            "street_number": {
                "id": "",
                "title": "",
                "location": {
                    "lat": 24.612911,
                    "lng": 118.046669
                },
                "_dir_desc": "东",
                "_distance": 68
            }
        },
        "formatted_addresses": {
            "rough": "集美区金砖未来创新园(诚毅北大街东50米)",
            "recommend": "集美区金砖未来创新园(诚毅北大街东50米)",
            "standard_address": "福建省厦门市集美区诚毅北大街与浦公山路交叉口东140米"
        }
    },
    "status": 0,
    "message": "query ok",
    "request_id": "089e0859-378a-45c7-b87c-1f066f25154f"
}
```

---------------------
- [hutool db文档](https://hutool.cn/docs/#/db/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%AE%80%E5%8D%95%E6%93%8D%E4%BD%9C-Db)
- [HikariCP数据库连接池](https://github.com/brettwooldridge/HikariCP)
