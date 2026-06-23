## 数字前面补位(0)
> 可用于生成规范的编码, 订单号, 序号等场景
```java
// AK-001
// AK-002
// AK-053
// AK-159
for (int i = 0; i < 200; i++) {
    // 保留3位数字, 不足前面补0
    String indexText = String.format("%03d", i);
    System.out.println("AK-" + indexText);
}
```

## 拆分字符串为List
```java
// 1. List.of, 2.CollUtil.newArrayList
String names = "chenzz, 嬴政, 白起, 廉颇, 李牧";
List<String> nameList = List.of(names.split(","));

nameList.forEach(x -> System.out.println("x = " + x));
log.warn("1. List列表 : {}", nameList);

log.info("2. List列表 : {}", CollUtil.newArrayList(names.split(",")));
```

## List拼接字符串
```java
// 1. Collectors.joining, 2.String.join
List<String> stringList = CollUtil.newArrayList("深圳", "广州", "浙江", "厦门", "北京");
String text = stringList.stream().collect(Collectors.joining(","));

String text2 = String.join(",", stringList);
// 深圳,广州,浙江,厦门,北京
System.out.println(text2);
```

## Function函数式接口的使用示例
> 只消费不需要返回值的可以使用`java.util.function.Consumer`
```java
import java.util.function.Function;
import java.util.function.BiFunction;

import java.util.List;
import java.util.Set;
import cn.hutool.core.lang.Dict;
import cn.hutool.db.Entity;

/**
 * 对象转换, 用户对象转换成Dict
 * eg:
 * Dict dict=funConvert.apply(t);
 */
private final Function<SysUser, Dict> funConvert = (oo) -> {
    Dict dict = Dict.create();

    int status = oo.getStatus();
    String avatar = oo.getAvatar();

    dict.set("avatar", avatar);
    dict.set("id", String.valueOf(oo.getUserId()));
    dict.set("loginName", oo.getLoginName());

    if (status == 1) {
        dict.set("tip", "用户账号已停用!");
    } else {
        dict.set("tip", "普通用户");
    }
    return dict;
};


/**
 * Entity转换成Dict(并转换成驼峰式字段名)
 * <p>
 * eg:
 * List<Dict> dictList = entityList.stream().map(funConvert).collect(Collectors.toList());
 */
private final Function<Entity, Dict> funConvert = (entity) -> {
    Set<String> fieldNames = entity.getFieldNames();
    Dict dtRow = Dict.create();
    fieldNames.forEach(tt -> {
        dtRow.set(StrUtil.toCamelCase(tt), entity.get(tt));
    });
    return dtRow;
};


/**
 * 根据关键词在List中查找符合条件的数据, 返回单条或者null
 * <p>
 * Dict dictQuery = funQuery.apply(list, "sn00005");
 */
private final BiFunction<List<Dict>, String, Dict> funQuery = (dictList, keyWords) -> {
    Optional<Dict> optionalDict = dictList.stream().filter(oo -> {
        String orderSn = oo.getStr("order_sn");
        return orderSn.equalsIgnoreCase(keyWords);
    }).findAny();
    // return optionalDict.isPresent() ? optionalDict.get() : null;
    return optionalDict.orElse(null);
};
```

## 使用Tuple实现在函数中返回多个值
> 参考python中元组tuple
```java
import java.util.function.Function;
import cn.hutool.core.lang.Tuple;

import cn.hutool.json.JSONArray;
import cn.hutool.json.JSONObject;
import cn.hutool.json.JSONUtil;
import java.util.*;

// Tuple的返回值顺序和生成的时候顺序对应
Function<String, Tuple> funParse = (jsonStr) -> {
    JSONObject jsonObject = JSONUtil.parseObj(jsonStr);

    Integer id = jsonObject.getInt("id");
    String title = jsonObject.getStr("title");
    JSONArray tagsJsonArr = jsonObject.getJSONArray("tags");
    List<String> tags = new ArrayList<>();

    for (var mm : tagsJsonArr) {
        if (mm instanceof String itemTag) {
            tags.add(itemTag);
        }
    }
    return new Tuple(id, title, tags);
};

// 使用, 通过Tuple的get获取返回值
String jsonText = """
        {
            "id": 1,
            "title": "Tuple-不可变数组类型(元组)",
            "content": "Tuple: 不可变数组类型（元组）, 用于多值返回 多值可以支持每个元素值类型不同, 元组是不可变的",
            "tags": [
                "java",
                "tuple",
                "函数",
                "返回多个值"
            ],
            "remark": "tuple用法"
        }
        """;

Tuple result = funParse.apply(jsonText);
Integer id = result.get(0);
String title = result.get(1);
List<String> tags = result.get(2);

Console.log("id(第一个)={}, title(第二个)={}, tags(第三个)={}", id, title, tags);
```

## 枚举Enum定义
```java
// ================1==================
package com.example.demo;

import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

/**
 * 枚举: 操作类型
 *
 * @author chenzz
 */
public enum OperateEnum {

    INSERT(1, "新增"),
    UPDATE(2, "修改"),
    DELETE(3, "删除"),
    QUERY(4, "查询"),
    CLEAN(5, "清理"),
    OTHER(6, "其他"),
    NO_USED(99, "--未使用--"),
    ;

    // -------------------------------------------------

    OperateEnum(int key, String desc) {
        this.key = key;
        this.desc = desc;
    }

    private final int key;
    private final String desc;

    public int getKey() {
        return key;
    }

    public String getDesc() {
        return desc;
    }

    /**
     * 根据key获取对应的枚举
     * <p>
     * eg: OperateEnum type = OperateEnum.getEnumByKey(5)
     */
    public static OperateEnum getEnumByKey(int key) {
        return Stream.of(OperateEnum.values()).filter(t -> t.key == key).findAny().orElse(null);
    }

    /**
     * 根据keys过滤枚举
     *
     * @param keys key集合
     * @return 枚举集合
     */
    public static List<OperateEnum> getEnumsByKeys(List<Integer> keys) {
        return Stream.of(OperateEnum.values()).filter(t -> keys.contains(t.key)).collect(Collectors.toList());
    }

    public static final int[] ARRAYS = Stream.of(values()).mapToInt(OperateEnum::getKey).toArray();
}


// ================2==================
import org.springframework.lang.Nullable;

import java.util.HashMap;
import java.util.Map;

/**
 * 请求方式
 *
 * @author chenzz
 */
public enum HttpMethod {
    GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE;

    private static final Map<String, HttpMethod> mappings = new HashMap<>(16);

    static {
        for (HttpMethod httpMethod : values()) {
            mappings.put(httpMethod.name(), httpMethod);
        }
    }

    @Nullable
    public static HttpMethod resolve(@Nullable String method) {
        return (method != null ? mappings.get(method) : null);
    }

    public boolean matches(String method) {
        return (this == resolve(method));
    }
}


// ================3==================
public enum ActionStatus {
    SUCCESS,
    FAIL,
}

// ================4==================
package com.example.enums;

import lombok.AllArgsConstructor;
import lombok.Getter;

/**
 * @author chenzz
 * 登录类型
 */
@Getter
@AllArgsConstructor
public enum LoginTypeEnum {
    WEIXIN_H5("weixin_h5", "微信h5"),
    H5("h5", "H5"),
    WECHAT("wechat", "公众号"),
    APP("app", "APP"),
    PC("pc", "pc网页"),
    ROUNTINE("routine", "小程序");

    private final String value;
    private final String desc;

}
```
