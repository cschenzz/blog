## 创建/生成Stream流
```java
// 集合List使用stream方法返回
// 1.根据可变参数/数组生成
Stream<String> stream1 = Stream.of("tom", "bill", "jackson", "john");
// 2.使用generate生成100-999的随机BigDecimal数据, 需要使用limit指定个数
Stream<BigDecimal> stream2 = Stream.generate(() -> RandomUtil.randomBigDecimal(BigDecimal.valueOf(100), BigDecimal.valueOf(999))).limit(10);
// 3.使用iterate迭代生成, 从1万开始, 生成999个, 步长为1的数据, 最后一个为10998
Stream<BigInteger> stream3 = Stream.iterate(BigInteger.valueOf(10000), n -> n.add(BigInteger.ONE)).limit(999);
// 4. 生成0-6共7个数字
Stream<Integer> stream4 = Stream.iterate(0, n -> n + 1).limit(7);
// 5. 生成1-12(包括头和尾)的整数
IntStream stream5 = IntStream.rangeClosed(1, 12);
// 打印输出*号组成的三角形
IntStream.range(0, 5).boxed().map(n -> cn.hutool.core.util.StrUtil.repeat("*", (n * 3) + 1)).forEach(System.out::println);

// 6. 使用IntStream生成5-9的字典列表
// 通过IntStream的boxed()方法返回Stream<Integer>类型
var randDictList = IntStream.rangeClosed(5, 9).boxed().map(n -> {
    Dict xxDict = Dict.create();

    xxDict.set("id", n);
    xxDict.set("name", RandomUtil.randomString(5));
    xxDict.set("age", RandomUtil.randomInt(18, 60, true, true));
    return xxDict;
}).toList();

stream1.forEach(tt -> System.out.println(tt));
stream2.forEach(System.out::println);
```

## 使用AtomicReference在forEach中赋值外部变量
```java
import cn.hutool.core.util.RandomUtil;

import java.math.BigDecimal;
import java.util.concurrent.atomic.AtomicReference;
import java.util.stream.IntStream;
import java.util.stream.Stream;

// 在lambda表达式中进行统计赋值
// 使用AtomicReference包装要统计的变量,使用set方法设置值, get方法获取值
// Variable used in lambda expression should be final or effectively final
AtomicReference<BigDecimal> refTotal = new AtomicReference<>(BigDecimal.ZERO);
Stream.generate(() -> RandomUtil.randomBigDecimal(BigDecimal.valueOf(100), BigDecimal.valueOf(999))).limit(10).forEach(oo -> {
    System.out.println(oo);
    refTotal.set(refTotal.get().add(oo));
});

// 总计
BigDecimal total = refTotal.get().setScale(2, BigDecimal.ROUND_UP);
System.out.println("总计:" + total);
```


## 基本操作
```java
// Arrays.asList
List<Integer> integerList = List.of(1, 2, 3, 4, 5, 6, 7, 6, 5, 4, 3, 2, 1);

// 过滤，收集所有偶数
List<Integer> collect = integerList.stream()
        .filter(t -> t % 2 == 0)
        // 在 peek 方法里面做任意没有返回值的事情，比如打印日志
        // foreach 属于终止方法, peek属于中间方法
        .peek(t -> System.out.println(t))
        // 下面的收集到list实现也可以使用Stream的toList(), 返回的是不可变集合, 列表内容不能添加和修改
        .collect(Collectors.toList());
System.out.println("过滤，收集所有偶数：" + collect);

int xxSum = integerList.stream().reduce(0, (a, b) -> a + b);
// int xxSum = integerList.stream().reduce(0, Integer::sum);
System.out.println("计算总和：" + xxSum);

BigDecimal totalPrice = CollUtil.newArrayList(BigDecimal.valueOf(199.9), BigDecimal.valueOf(289.0), BigDecimal.valueOf(399)).stream()
    .reduce(BigDecimal.ZERO, (a, b) -> a.add(b));
// 887.9
System.out.println("计算总和：" + totalPrice);
// 以上reduce可简写成
// reduce(BigDecimal.ZERO, BigDecimal::add)
// --------------------------


// 计算数量, 8
long count = integerList.stream().filter(oo -> oo < 5).count();
System.out.println("比5小的数字有(个): " + count);

Optional<Integer> collectX1 = integerList.stream().max(Comparator.comparingInt(x -> x));
if (collectX1.isPresent()) {
    System.out.println("求最大值：" + collectX1.get());
}

// 分组
// List<Dict> dictList = null;
// Map<Long, List<Dict>> xxCollect = dictList.stream().collect(Collectors.groupingBy(oo -> oo.getLong("uid")));

Map<Integer, List<Integer>> collectX2 = integerList.stream().collect(Collectors.groupingBy(Integer::intValue));
System.out.println("分组：" + collectX2);

// 中间管道: filter, map, flatMap, limit, skip, concat, distinct, sorted, peek
// 终止管道: count, max, min, findFirst, findAny, anyMatch, allMatch, noneMatch, collect, toArray, iterator, forEach
// 中间操作(返回值都是Stream)可链式多个操作, 终止操作放最后执行, 只能有一个

// forEach和forEachOrdered, 只有在parallel()并行处理情况下, 才会有差别
// forEach在并行情况下, 自由执行, 不考虑顺序
// forEachOrdered在并行情况下, 按顺序执行

// --------------------------
// Optional示例
// 1. Optional链式获取深层次对象结构的值, map用法
Dict dict = Dict.create().set("user", Dict.create().set("name", "tom").set("age", 99)).set("roles", List.of("admin", "sales"));
// {"user":{"name":"tom","age":99},"roles":["admin","sales"]}
Optional<Dict> optionalDict = Optional.ofNullable(dict);
// 结果: tom, 使用Optional的map一级一级获取深层数据值
String name = optionalDict
        // 前面一步的map结果会作为下一步map的输入参数, 并且在任何一个map中如果返回值为null, 则结果使用else中定义的值
        .map(oo -> oo.get("user", Dict.create()))
        .map(oo -> oo.getStr("name"))
        .orElse("-");

// 2. ifPresent用法
optionalDict.ifPresent(mm -> {
    List<String> roles = mm.get("roles", Collections.emptyList());
    // [admin, sales]
    System.out.println(roles);
});
```

## Optional读取json示例
```java
String jsonText = """
        {"user":{"name":"tom","age":99,"location":{"lat":24.56,"lng":118.06}},"roles":["admin","sales"],"uid":8}
        """;

JSONObject tmpJsonObject = new JSONObject(jsonText);

// 使用optional的filter和map过滤并获取层次比较深的数据并在获取到数据后做处理(ifPresent, ifPresentOrElse)
Optional.of(tmpJsonObject)
        // .filter(oo -> oo.getInt("uid") > 1)
        .map(oo -> oo.getJSONObject("user"))
        .filter(oo -> oo.getInt("age") > 20)
        .map(oo -> oo.getJSONObject("location"))
        .map(oo -> oo.getDouble("lat"))
        .ifPresentOrElse(lat -> {
            Console.log("纬度(lat)={}", lat);
        }, () -> {
            Console.error("未获取到纬度(lat)数据");
        });

// 99
int tmpAge = Optional.ofNullable(tmpJsonObject)
        .map(oo -> oo.getJSONObject("user"))
        .map(oo -> oo.getInt("age"))
        .orElse(18);

// 24.56
double tmpLat = Optional.ofNullable(tmpJsonObject)
        .map(oo -> oo.getJSONObject("user"))
        .map(oo -> oo.getJSONObject("location"))
        .map(oo -> oo.getDouble("lat"))
        .orElse(0.0);
// Optional.of: 参数传null会抛出错误
// Optional.ofNullable: 参数传null返回空Optional
```

## java9后新添功能
```java
// takeWhile, 从Stream中依次获取满足条件的元素，直到不满足条件为止结束获取
// dropWhile, 从Stream中依次删除满足条件的元素，直到不满足条件为止结束删除
// 以上均为中间管道操作. 注意: 只要返回为Stream的都是中间管道, 其余为终止管道

// ------------------------------------
// x -> x % 2 == 0 ，判断是否为偶数
// 结果为: 12, 4, 即使后面还有6, 8但因为前面有3不符合条件, 后面的直接不处理
int[] intArr = {12, 4, 3, 6, 8, 9};
IntStream.of(intArr).takeWhile(x -> x % 2 == 0).forEach(System.out::println);
// 3, 6, 8, 9， 根据上面把符合条件的剔除
IntStream.of(12, 4, 3, 6, 8, 9).dropWhile(x -> x % 2 == 0).forEach(System.out::println);
```

## 字符串拼接
```java
String[] strs = {"java8", "is", "easy", "to", "use"};

String strX13 = Stream.of(strs).collect(Collectors.joining(" "));
System.out.println("1.字符串拼接：" + strX13);

String strText = String.join(",", strs);
System.out.println("2.字符串拼接：" + strText);

// --------------------------------------
import java.util.stream.Stream;
import cn.hutool.core.lang.Console;
import cn.hutool.core.lang.PatternPool;
import cn.hutool.core.util.ReUtil;

// flatMap处理
List<String> listStr = Stream.of(strs).peek(Console::log)
        // flatMap需要返回Stream包装的对象, 在flatMap函数里stream对象可以使用stream的中间管道操作(如filter, map, peek等)
        // 注意flatMap和map返回的都是Stream, 但函数里面返回值的写法不一样
        // eg. 这里测试返回仅包含字母的字符串
        .flatMap(oo -> Stream.of(oo).filter(tt -> ReUtil.isMatch(PatternPool.WORD, tt)))
        .toList();

// ["is","easy","to","use"]
Console.log("{}", JSONUtil.toJsonStr(listStr));
```

## 找出List中的重复数据
```java
import java.util.*;
import java.util.function.Function;
import java.util.stream.Collectors;

List<String> list = List.of("A", "B", "C", "A", "D", "B");

var frequencyMap = list.stream()
        .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
// {A=2, B=2, C=1, D=1}
Console.log("{}", frequencyMap);

var duplicates = frequencyMap.entrySet().stream()
        .filter(entry -> entry.getValue() > 1)
        .map(Map.Entry::getKey)
        .toList();
// 重复数据：[A, B]
Console.log("重复数据：{}", duplicates);
```

## filter过滤, stream分页
```java
import java.util.Optional;
import cn.hutool.core.util.NumberUtil;

List<Integer> integerList = List.of(1, 2, 3, 4, 5, 6, 7, 6, 5, 4, 3, 2, 1);

// 返回满足条件的第一个元素, findAny也可以使用Optional包装
// filter可以多次使用(多条件过滤), 过滤后再进行过滤(例如>3并且是质数)
Optional<Integer> first = integerList.stream().filter(t -> t > 3).filter(NumberUtil::isPrimes).findFirst();
if (first.isPresent()) {
    // 结果为5
    System.out.println("满足条件的第一个元素:" + first.get());
}

// findAny相对于findFirs区别在于findAny不一定返回第一个而是返回任意一个
// 实际上对于顺序流式处理而言, findFirst和findAny返回的结果是一样的
// 至于为什么会这样设计, 当我们启用并行流式处理的时候, 查找第一个元素往往会有很多限制, 如果不是特别需求
// 在并行流式处理中使用findAny的性能要比findFirst好

// .distinct()为可选操作, 使用Optional的orElse在没有值的时候设置一个默认值
Integer any1 = integerList.stream().filter(t -> t > 100).distinct().findAny().orElse(-100);
System.out.println("any1：" + any1);

// -------------------------------
// 使用stream实现分页,注意先skip再limit
// 每页数据条数
int pageSize = 5;
// 页码, 从1开始
int pageIndex = 2;
List<Integer> pageList = integerList.stream().skip((pageIndex - 1) * pageSize).limit(pageSize).collect(Collectors.toList());
```

## java8流操作示例
```java
import cn.hutool.core.collection.CollUtil;
import cn.hutool.core.lang.Console;
import cn.hutool.core.lang.Dict;
import cn.hutool.json.JSONUtil;

import java.math.BigDecimal;
import java.util.Comparator;
import java.util.List;
import java.util.Optional;
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

List<Dict> dictList = CollUtil.newArrayList(
        // ?
        // BigDecimal.valueOf作用相当于new BigDecimal(String s), 可以精确计算
        // sku, 购买数量, 库存, 单价
        Dict.create().set("sku", "a").set("buy", 9).set("stock", 1000).set("price", BigDecimal.valueOf(79.0)),
        Dict.create().set("sku", "b").set("buy", 7).set("stock", 50).set("price", BigDecimal.valueOf(89.0)),
        Dict.create().set("sku", "c").set("buy", 25).set("stock", 60).set("price", new BigDecimal("99.0"))
);

// Collectors.toMap, {"a":"库存:1000","b":"库存:50"}
// Map<String, String> mapsSku = dictList.stream().filter(oo -> oo.getInt("buy") < 10).collect(Collectors.toMap(oo -> oo.getStr("sku"), oo -> "库存:" + oo.getInt("stock")));

// ---------------------------
// 1.根据条件过滤出单条记录(有多条也仅返回一条), 使用Optional包装
Optional<Dict> optionalDict = dictList.stream().filter(oo -> {
    int stock = oo.getInt("stock");
    BigDecimal price = oo.getBigDecimal("price");

    // 返回价钱>90, 库存>50的商品
    return price.compareTo(BigDecimal.valueOf(90)) > 0 && stock > 50;
}).findAny();

optionalDict.ifPresent(mm->{
    Console.log("------存在------\r\n{}", JSONUtil.toJsonPrettyStr(mm));
});
if (optionalDict.isPresent()) {
    Dict tempDict = optionalDict.get();
    Console.log("------1.存在记录(功能同上)------\r\n{}", JSONUtil.toJsonPrettyStr(tempDict));
}

// filter, findAny查找记录存在一把梭
dictList.stream().filter(oo -> {
    String ooSku = oo.getStr("sku");
    return "c".equals(ooSku);
}).findAny().ifPresentOrElse(mm -> {
    Console.log("---debug.5.{}", mm);
}, () -> {
    Console.log("---debug.5.不存在!");
});
// -------------------------------

// 直接对list进行排序(从大到小)
// dictList.sort((o1, o2) -> {
//     int x1 = o1.getInt("stock");
//     int x2 = o2.getInt("stock");
//
//     // 从小到大Integer.compare(x1, x2)
//     return Integer.compare(x2, x1);
// });
// -------------------------------

// 2.根据库存从大到小排序, 生成新的list对象
List<Dict> dictListSorted = dictList.stream().sorted((o1, o2) -> {
    int x1 = o1.getInt("stock");
    int x2 = o2.getInt("stock");
    // 在这里可以多条件排序, 根据compare结果为0(表示相等)再进行其他条件的排序
    // 从小到大Integer.compare(x1, x2)
    return Integer.compare(x2, x1);
}).collect(Collectors.toList());
System.out.println("2.根据库存从大到小排序:" + dictListSorted);
// 根据价钱升序排列
// var list = dictList.stream().sorted(Comparator.comparing(o -> o.getBigDecimal("price"))).toList();
// 根据库存降序排列
// var list = dictList.stream().sorted(Comparator.comparing(o -> o.getInt("stock"), Comparator.reverseOrder())).toList();
// personList.stream().sorted(Comparator.comparingInt(Person::getAge).thenComparing(Person::getSalary)).collect(Collectors.toList());

// Optional.ofNullable-构造方法、map-逐层安全地拆解value、filter-过滤值、orElse/orElseThrow-最终返回、stream-转为流
// BigDecimal tmpSalary = Optional.ofNullable(tmpPerson).map(Person::getSalary).orElse(BigDecimal.ZERO);
// ------------------------------

// 3.找出价钱最便宜的商品
Optional<Dict> min = dictList.stream().min((o1, o2) -> {
    BigDecimal x1 = o1.getBigDecimal("price");
    BigDecimal x2 = o2.getBigDecimal("price");
    return x1.compareTo(x2);
    // return Integer.compare(x1, x2);
});
if (min.isPresent()) {
    Console.log("3.最便宜的商品: {}", min.get());
}
// 以下写法为上面计算的简写(只是改成了找最贵的商品)
Optional<Dict> max = dictList.stream().max(Comparator.comparing(o -> o.getBigDecimal("price")));
// ------------------------------

// 4.根据sku分组
Map<String, List<Dict>> groupCollect = dictList.stream().collect(
        Collectors.groupingBy(oo -> {
            String sku = oo.getStr("sku");
            return sku;
        })
);
System.out.println("4.根据sku分组:" + groupCollect);
// 分组求和
// 根据sku分组并计算购物消费金额
// Map<String, Integer> salarySumMap = personList.stream().collect(Collectors.groupingBy(Person::getSex, Collectors.summingInt(Person::getSalary)));
Map<String, Double> skuCostMap = dictList.stream().collect(Collectors.groupingBy(oo -> oo.getStr("sku"), Collectors.summingDouble(tt -> {
            // 单价X数量, 并计算对应sku总价(sum和)
            BigDecimal xxx = tt.getBigDecimal("price").multiply(tt.getBigDecimal("buy"));
            return xxx.doubleValue();
        }
)));
System.out.println("4.sku分组消费金额:" + skuCostMap);
// 按照分组消费金额map中value的数量逆序打印每个entry
skuCostMap.entrySet().stream().sorted(Map.Entry.<String, Double>comparingByValue().reversed())
        .forEachOrdered(System.out::println);
System.out.println("-----------------------");
// Map<String, Set<String>> orderMap = orderList.stream().collect(Collectors.groupingBy(Order::getSku, Collectors.mapping(Order::getSkuName, Collectors.toSet())));
// ------------------------------

// 5.使用规约reduce计算价格总和
BigDecimal sumTotal = dictList.stream()
        .map(xx -> {
            // 单价X数量
            BigDecimal xox = xx.getBigDecimal("price").multiply(xx.getBigDecimal("buy"));
            return xox;
        })
        .reduce(BigDecimal.ZERO, (a, b) -> {
            Console.log("a={}, b={}", a, b);
            // BigDecimal a1 = new BigDecimal(String.valueOf(a));
            // BigDecimal b1 = new BigDecimal(String.valueOf(b));
            return a.add(b);
        });
System.out.println("5.购物总花费:" + sumTotal);

// 6.使用flatMap实现过滤+映射
var tempYy = dictList.stream()
        .flatMap(oo -> Stream.of(oo).filter(xx -> xx.getInt("stock") > 100).map(xx -> xx.getStr("sku")))
        .findAny();
// 存在和不存在分别处理(参考代码段1): void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)
// 存在: void ifPresent(Consumer<? super T> action)
tempYy.ifPresent(oo -> {
    // 打印6.a
    log.info("6.{}", oo);
});

// 7.计算count, sum, min, max值
// 示例为对于List列表中某一个字段的统计数据(必须能转换成int, long, double数值类型才可以进行统计)
// summaryStatistics1: {"count":3,"sum":267,"min":79,"max":99}
var summaryStatistics1 = dictList.stream().mapToDouble(oo -> oo.getBigDecimal("price").doubleValue()).summaryStatistics();
var summaryStatistics2 = dictList.stream().mapToInt(oo -> oo.getInt("buy")).summaryStatistics();
```
> 参考以下flatMap使用案例
* 🍊[使用flatMap在json中查找满足条件的项](/backend/java/hutool-java-2.md#在json中查找列表中满足条件的项指定用户的目标值)


-------------------

- [吃透JAVA的Stream流操作，多年实践总结](https://juejin.cn/post/7118991438448164878)
- [讲透JAVA Stream的collect用法与原理，远比你想象的更强大](https://juejin.cn/post/7121539527151190053)
