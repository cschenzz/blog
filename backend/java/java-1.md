## 声明日志打印变量
```java
// hutool log util
import cn.hutool.log.Log;
import cn.hutool.log.LogFactory;
private static final Log log = LogFactory.get();


import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
private final Logger logger = LoggerFactory.getLogger(this.getClass());
```

## BigDecimal示例
```java
// add加, subtract减, multiply乘, divide除
BigDecimal a = BigDecimal.valueOf(10);
BigDecimal b = new BigDecimal("3");

// HALF_UP: 5入, UP: 往上入, DOWN: 往下直接舍弃(取整时可用)
// 2数相除, 保留2位小数(5入), 3.33
BigDecimal result = a.divide(b, 2, RoundingMode.HALF_UP);
log.info("两数相除[{}/{}]保留2位小数(5入)={}", a, b, result);

// 3.1416, BigDecimal的setScale会返回新对象, 原BigDecimal值不会变
BigDecimal b2 = BigDecimal.valueOf(3.1415926535);
log.info("{}保留4位小数:{}", b2, b2.setScale(4, RoundingMode.HALF_UP));

// 3, setScale向下取整
log.info("BigDecimal取整:{}", b2.setScale(0, RoundingMode.DOWN));

// BigDecimal比较大小, 不能使用equals和==比较
// 使用BigDecimal的compareTo根据返回值来进行比较
// -1, 小于
int result1 = BigDecimal.valueOf(99).compareTo(BigDecimal.valueOf(100));

// 1, 大于
int result2 = BigDecimal.valueOf(99.99).compareTo(BigDecimal.valueOf(99));

// 0, 相等
int result3 = BigDecimal.valueOf(9.9).compareTo(BigDecimal.valueOf(9.9));
```

## BigDecimal格式化输出
```java
import cn.hutool.core.util.NumberUtil;

// 四舍五入2位小数, 899566.18
var txtPrice1 = NumberUtil.decimalFormat(".00", BigDecimal.valueOf(899566.17838));

// 整数每三位以逗号进行分隔, 999,888,777,666,555,444
var txtPrice2 = NumberUtil.decimalFormat(",###", BigDecimal.valueOf(999888777666555444L));

// 以百分比方式计数，并取两位小数, 95.26%
var txtPrice3 = NumberUtil.decimalFormat("#.##%", BigDecimal.valueOf(0.95259));
```

## switch表达式使用
```java
// 1. 根据类型获取说明
int type = 1;
var typeText = switch (type) {
    case 1 -> "单选题";
    case 2 -> "多选题";
    case 3 -> "判断题";
    default -> "神仙题";
};

// 2. 根据文本获取boolean结果
String valueStr = "ok";
var boolResult = switch (valueStr) {
    case "true", "yes", "ok", "1" -> true;
    case "false", "no", "0" -> false;
    default -> false;
};
```

## java8时间使用
```java
// time: 2023-01-02 14:25:59
LocalDateTime time = LocalDateTime.of(2023, 1, 2, 14, 25, 59);
// 计算24小时后时间
LocalDateTime time2 = time.plusHours(24);
// time2: 2023-01-03 14:25:59
Console.log("time2: {}", LocalDateTimeUtil.format(time2, "yyyy-MM-dd HH:mm:ss"));

// ---------------------------
// Date->LocalDateTime
// LocalDateTime dateTimeNow = LocalDateTime.ofInstant(new Date(), ZoneId.systemDefault());
// ---------------------------
LocalDateTime now = LocalDateTime.now();
// LocalDateTime->Date
Date dateNow = Date.from(now.atZone(ZoneId.systemDefault()).toInstant());

String nowText = LocalDateTimeUtil.format(now, "yyyy-MM-dd HH:mm:ss");
log.info("now={}", nowText);

LocalDateTime dayBefore7 = now.plusDays(-7);
log.info("7天前: {}", LocalDateTimeUtil.format(dayBefore7, "yyyy-MM-dd HH:mm:ss"));

// hutool时间工具计算2个日期时间差LocalDateTimeUtil.between()
// 10天后时间
LocalDateTime dayAfter10 = now.plusDays(10);

Duration duration = Duration.between(dayBefore7, dayAfter10);
log.info("计算间隔时间(天): {}", duration.toDays());

log.info("格式化时间: {}", dayAfter10.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));

```


## Optional使用
```java
package com.example.test;

import lombok.Data;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import java.util.Date;
import java.util.Optional;

public class TempX01Tests {

    @DisplayName("Optional测试获取深层次对象的值")
    @Test
    void testX01() {
        User user1 = new User();
        user1.setId(100);
        user1.setUsername("czz");

        LoginUser loginUser1 = new LoginUser();
        loginUser1.setToken("token-1");
        loginUser1.setObjUser(user1);
        // loginUser1.setUser(user1);
        // loginUser1.setObjUser(888);

        var xValue = Optional.of(loginUser1)
                .map(LoginUser::getObjUser)
                .map(oo -> {
                    if (oo instanceof String) {
                        return oo.toString();
                    } else if (oo instanceof User user) {
                        return user.getId();
                    }
                    System.out.println("----1.");
                    return null;
                })
                .orElse("-");

        System.out.println(xValue.getClass());
        System.out.println(xValue);
    }


    /**
     * 登陆用户
     */
    @Data
    static class LoginUser {
        private String token;
        private Date loginTime;
        private User user;
        private Object objUser;
    }

    /**
     * 用户信息
     */
    @Data
    static class User {
        private int id;
        private String username;
        private String password;
        private int age;
    }


}
```



## List元素删除测试

```java
List<Dict> list = CollUtil.newLinkedList(
        Dict.create().set("id", 1).set("name", "葫芦娃").set("age", 7),
        Dict.create().set("id", 2).set("name", "孙大圣").set("age", 999),
        Dict.create().set("id", 3).set("name", "太上老君").set("age", 99999)
);
log.info("1.dictList: {}", list.size());
// ===========================================
Iterator<Dict> iterator = list.iterator();
while (iterator.hasNext()) {
    Dict next = iterator.next();
    // 根据条件删除元素
    if (next.getInt("age") < 100) {
        iterator.remove();
    }
}
log.info("2.dictList: {}", list.size());
```


## 判断一个List中是否存在指定元素
```java
List<Dict> dictList = CollUtil.newArrayList(
        Dict.create().set("no", "b0002").set("name", "tim"),
        Dict.create().set("no", "c0002").set("name", "tom"),
        Dict.create().set("no", "x0001").set("name", "xxx")
);
// 1.根据条件判断列表是否存在指定元素
boolean exist = dictList.stream().filter(o ->
        o.getStr("name").equals("tom")
).count() > 0;
// 使用anyMatch实现同样功能
// boolean exist = dictList.stream().anyMatch(o -> o.getStr("name").equals("tom"))
// 其他判断可参考: anyMatch, allMatch, noneMatch
log.info("dictList是否存在tom:{}", exist);

// 2.使用IntStream实现判断集合中是否存在某个数, 指定数据是否在集合中
Integer xx = 4096;
long existCount = IntStream.of(5, 8, 9, 4096).filter(o -> o == xx).count();
// 结果true
log.info("exist={}", existCount > 0);
// boolean exist = IntStream.of(5, 8, 9, 4096).anyMatch(oo -> oo == xx);

// 3.判断状态是否是其中一种
// exist = true, 99999也存在集合中(集合中数值在这里为Integer类型)
boolean exist = CollUtil.newArrayList(0, 1, 2, 3, 4, 99999).contains(3);
```

## map遍历方式
```java
@DisplayName("map遍历")
@Test
void testX01() {
    Map<String, String> map = new LinkedHashMap<>();
    map.put("name", "czz");
    map.put("age", "88");
    map.put("sex", "男");
    map.put("email", "czz@qq.com");
    // key
    for (String key : map.keySet()) {
        log.debug(key);
    }
    // value
    for (String value : map.values()) {
        log.debug(value);
    }
    // k,v
    for (var mmx : map.entrySet()) {
        log.info("{}, k={}, v={}", mmx, mmx.getKey(), mmx.getValue());
    }

    // map遍历-1
    map.entrySet().forEach(t -> log.debug("{}: {}", t.getKey(), t.getValue()));
    // map遍历-2
    map.forEach((k, v) -> {
        log.debug("{}={}", k, v);
    });
}
```

------------------------
- [Java中泛型详解](https://blog.csdn.net/weixin_45395059/article/details/126006369)
