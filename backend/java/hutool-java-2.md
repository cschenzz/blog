## 常用类说明
* ☕JSON类: JSON对象-JSONObject, JSON数组-JSONArray
* 🌰JSONUtil
  * toJsonStr(对象转换为JSON字符), toJsonPrettyStr(转为JSON字符串并格式化)
  * isTypeJSON(是否为JSON类型字符串，首尾都为大括号或中括号判定为JSON字符串)
  * parseObj(JSON字符串转JSONObject对象), parseArray(JSON字符串转JSONArray)
  * toBean(JSON字符串转为实体类对象), toList(将JSONArray字符串转换为Bean的List，默认为ArrayList)
  * toXmlStr(转换为XML字符串), readJSONObject(从文件中读取json)



> JSON字符串返回前端处理
```java
// json字符串转换成JSONObject, 默认为true(忽略null值), false不忽略null值
cn.hutool.json.JSONObject jsonObject = cn.hutool.json.JSONUtil.parseObj(jsonText, false);

// 通过查看JSONUtil.wrap代码: jsonConfig.isIgnoreNullValue() ? null : JSONNull.NULL
// 可以发现, 当json值为null时, 这里对null进行了转换, 转换成了JSONNull.NULL, 但是这在使用接口返回到前端时就会有问题
// 因为spring boot需要使用jackJson做序列化后再返回到前端, 但是JSONNull数据类型在jackJson中不支持而无法序列化, 导致出错. 
// 建议使用alibaba的fastjson2来把json文本转换成fastjson2对应的JSONObject, 这里默认是不忽略null值(即保留null值)
// 然后可以把此alibaba的JSONObject对象put到Map对象然后返回给前端, 这样就可以解决jackJson序列化错误的问题

// --------------------------------------------
// 返回前端json对象建议用fastjson2, 用hutool json对于null数据会发生序列化错误
// No serializer found for class cn.hutool.json.JSONNull and no properties discovered to create BeanSerializer
// HuTool的JSON里用JSONNull对象代替了null，而且JSONNull类没有适用于Jackson序列化的序列化器。
// 如果接口返回值对象里使用了HuTool的JSON，又凑巧存在被JSONNull对象代替了的null值，则序列化时会出现以上报错。
com.alibaba.fastjson2.JSONObject jsonObject = com.alibaba.fastjson2.JSONObject.parseObject(jsonText);
```

## JSONUtil使用示例
```java
import cn.hutool.json.JSONArray;
import cn.hutool.json.JSONConfig;
import cn.hutool.json.JSONObject;
import cn.hutool.json.JSONUtil;

Dict dict = Dict.create()
        .set("tags", List.of("后端", "java", "stream", "集合"))
        .set("text", "JAVA Stream的collect用法与原理")
        .set("author", Dict.create().set("userId", 1).set("userName", "czz").set("avatar", "http://xxx.com/xx.png"))
        .set("price", 59.0)
        .set("create_time", null)
        .set("id", 100L);

// 对象转换成json字符串, 默认忽略null值
String jsonStr = JSONUtil.toJsonStr(dict);
// 使用JSONConfig配置json不忽略null值, 小数不去除末尾多余的0
String jsonText = JSONUtil.toJsonStr(dict, JSONConfig.create().setIgnoreNullValue(false).setStripTrailingZeros(false));
log.info("字符串json: {}, {}", jsonStr, jsonText);

// json字符串转换成字典
Dict beanDict = JSONUtil.toBean(jsonStr, Dict.class);
log.info("字典获取text: {}", beanDict.getStr("text"));

// json字符串转换成JSONObject
JSONObject jsonObject = JSONUtil.parseObj(jsonStr);
log.info("JSONObject获取id: {}", jsonObject.getLong("id"));
String authorAvatar = jsonObject.getByPath("author.avatar", String.class);
log.info("根据path获取avatar: {}", authorAvatar);

// 输出xml
String xmlStr = JSONUtil.toXmlStr(jsonObject);
log.info("JSONObject转换成xml: {}", xmlStr);
```


## JSONObject使用
```java
Person tmpPerson = new Person("孙悟空", 99);
Dict tmpDict = Dict.create()
        .set("address", "花果山水帘洞")
        .set("skills", List.of("金箍棒", "筋斗云"));

JSONObject jsonObject = new JSONObject();
jsonObject.set("uid", 1);
jsonObject.set("person", tmpPerson);
jsonObject.set("extra_config", tmpDict);

// {"uid":1,"person":{"name":"孙悟空","age":99},"extra_config":{"address":"花果山水帘洞","skills":["金箍棒","筋斗云"]}}
Console.log(jsonObject.toString());
Console.log(jsonObject.toStringPretty());
```


## json数组解析
```java
// Java 17 新特性：文本块
String jsonText = """
        {
            "userId": 1,
            "userType": 10,
            "name": "chenzz",
            "nickName": "陈某",
            "remark": "用户分模块权限配置",
            "authConfig": [
                {
                    "module": "yd-customer",
                    "tip": "yd客户列表模块",
                    "filterType": "where",
                    "dbField": "客户经理",
                    "op": "in",
                    "value": "李白,凯皇,孙猴子"
                },
                {
                    "module": "daily-paper",
                    "tip": "日报模块",
                    "filterType": "where",
                    "dbField": "user_id",
                    "op": "in",
                    "value": "1,3,5"
                }
            ]
        }
        """;
JSONObject jsonObject = JSONUtil.parseObj(jsonText);
JSONArray array = jsonObject.getJSONArray("authConfig");

// 找出日报模块的权限配置
JSONObject jsonObjectAuthConfigX01 = null;
for (var mm : array) {
    // instanceof的模式匹配
    if (mm instanceof JSONObject jsonObjectItem) {
        String module = jsonObjectItem.getStr("module");
        log.info("module={}", module);
        if ("daily-paper".equals(module)) {
            // 找到符合条件记录
            jsonObjectAuthConfigX01 = jsonObjectItem;
        }
    }
}

if (jsonObjectAuthConfigX01 != null) {
    String tip = jsonObjectAuthConfigX01.getStr("tip");
    String filterType = jsonObjectAuthConfigX01.getStr("filterType", "all");
    String dbField = jsonObjectAuthConfigX01.getStr("dbField");
    String op = jsonObjectAuthConfigX01.getStr("op");
    String value = jsonObjectAuthConfigX01.getStr("value", "");
    log.info("日报模块的权限配置, tip={}, filterType={}, dbField={}, op={}, value={}", tip, filterType, dbField, op, value);
}
// --------------------------------

// 使用stream中flatMap, filter实现json查询(结果同上), findAny返回单个
var jsonObjectXx = array.stream().flatMap(oo -> {
    if (oo instanceof JSONObject jsonObjectItem) {
        return Stream.of(jsonObjectItem).filter(xx -> "daily-paper".equals(xx.getStr("module")));
    }
    return Stream.of(new JSONObject());
}).findAny();

// 输出: tip=日报模块
jsonObjectXx.ifPresent(oo -> {
    log.info("1.{}", oo);
    log.info("tip={}", oo.getStr("tip", ""));
});
```

## 使用BiFunction解析json并返回符合条件的JSONObject对象
```java
// import java.util.function.BiFunction;
// BiFunction: FunctionalInterface, 2个入参, 1个返回值
// 功能同上
BiFunction<String, String, JSONObject> biFunction = (jsonStr, findModule) -> {
    JSONObject jsonObject = JSONUtil.parseObj(jsonStr);
    JSONArray array = jsonObject.getJSONArray("authConfig");

    // 1. 使用for循环实现
    // 返回的变量
    JSONObject jsonObjectReturn = null;
    for (var mm : array) {
        // instanceof的模式匹配
        if (mm instanceof JSONObject jsonObjectItem) {
            String tempModule = jsonObjectItem.getStr("module");
            if (findModule.equals(tempModule)) {
                // 找到符合条件记录
                jsonObjectReturn = jsonObjectItem;
            }
        }
    }
    return jsonObjectReturn;

    // 2. 使用stream的flatMap实现 
    // -------------------------
    // -------------------------
    // var jsonObjectXx = array.stream().flatMap(oo -> {
    //     if (oo instanceof JSONObject jsonObjectItem) {
    //         return Stream.of(jsonObjectItem).filter(xx -> findModule.equals(xx.getStr("module")));
    //     }
    //     return Stream.of(new JSONObject());
    // }).findAny();
    // -------------------------
    // return jsonObjectXx.isPresent() ? jsonObjectXx.get() : null;
    // 可以简写成下面的形式
    // return jsonObjectXx.orElse(null);
    // =========================
};

JSONObject jsonObjectAuthConfigX01 = biFunction.apply(jsonText, "daily-paper");

// tip=日报模块
if (jsonObjectAuthConfigX01 != null) {
    String tip = jsonObjectAuthConfigX01.getStr("tip", "");
    log.info("tip={}", tip);
} else {
    log.error("没有找到!");
}
```

## 在json中查找列表中满足条件的项(指定用户的目标值)
```java
String jsonText = """
        {
            "config": [
                {
                    "name": "李白",
                    "goal": 50000
                },
                {
                    "name": "夏侯惇",
                    "goal": 45000
                },
                {
                    "name": "王昭君",
                    "goal": 30000
                },
                {
                    "name": "明世隐",
                    "goal": 90000
                }
            ],
            "quarter": "2024-1",
            "quarterText": "2023年Q4季度(2024-01到2024-03)P4P消耗目标设置"
        }
        """;

// json文本转化成Dict字典
Dict xxDict = JSONUtil.toBean(jsonText, Dict.class);
// System.out.println(JSONUtil.toJsonPrettyStr(xxDict));


JSONArray configArray;
if (JSONUtil.isTypeJSONObject(jsonText)) {
    JSONObject jsonObject = JSONUtil.parseObj(jsonText);
    configArray = jsonObject.getJSONArray("config");
} else {
    configArray = new JSONArray();
}

// -----------^o^-------------
String tmpName = "明世隐";
var tmpGoal = configArray.stream().flatMap(oo -> {
    if (oo instanceof JSONObject jsonObjectItem) {
        return Stream.of(jsonObjectItem).filter(xx -> tmpName.equals(xx.getStr("name"))).map(xx -> xx.getBigDecimal("goal"));
    }
    return Stream.of(BigDecimal.ZERO);
}).findAny();
// 没有则默认为10
BigDecimal tmpXxQuota = tmpGoal.orElse(BigDecimal.TEN);
Console.log("{}目标/指标为:{}", tmpName, tmpXxQuota);
```

---------------------
- [hutool 文档](https://hutool.cn/docs/index.html)
- [fastjson v2](https://github.com/alibaba/fastjson2) | [gitee](https://gitee.com/wenshao/fastjson2) | [使用](https://blog.csdn.net/qq_33697094/article/details/128114939)
