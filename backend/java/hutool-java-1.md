## hutool常用工具类
包括

1. 转换类, 日期时间类, Io文件流
2. 工具类: 字符串工具-StrUtil, 16进制工具-HexUtil, 对象工具-ObjectUtil, 命令行工具-RuntimeUtil, 随机工具-RandomUtil, 压缩工具-ZipUtil, 正则工具-ReUtil, 信息脱敏工具-DesensitizedUtil
3. 常用类: HashMap扩展-Dict
4. 集合类: 集合工具-CollUtil, Map工具-MapUtil
5. 可复用字符串生成器-StrBuilder
6. 自定义线程池-ExecutorBuilder, 高并发测试-ConcurrencyTester
7. 图片工具-ImgUtil, 网络工具-NetUtil, URL生成器-UrlBuilder
8. JSON工具-JSONUtil, JSON对象-JSONObject, JSON数组-JSONArray
9. 摘要加密-Digester, 签名和验证-Sign, 国密算法工具-SmUtil
10. 数据库简单操作-Db
11. Http客户端工具类-HttpUtil
12. 简易Http服务器-SimpleServer
13. 二维码工具-QrCodeUtil
14. Excel大数据生成-BigExcelWriter
15. JWT工具-JWTUtil


## 常用类说明
* 🍄RuntimeUtil
  * randomEleList: 随机获得列表中的一定量的元素，此方法与randomEles(List, int) 不同点在于，不会获取重复位置的元素
  * randomEleSet: 随机获得列表中的一定量的不重复元素，返回Set

* 🍑CollUtil
  * union, unionAll(并集), intersection(交集), 差集(disjunction), subtract(集合的单差集)

* 🐶NumberUtil
  * round(四舍五入, 保留固定位数小数), add(多个数相加, null会当作0处理), isNumber(是否为数字)
  * decimalFormat(格式化BigDecimal, double, 对 DecimalFormat 做封装)
  * [BigDecimal格式化参考](/backend/java/java-1.md#bigdecimal格式化输出)
* DataSizeUtil: 数据大小工具类, format格式化数据大小字符串, 如5M, 12Kb
* [计时器工具-TimeInterval](https://hutool.cn/docs/index.html#/core/%E6%97%A5%E6%9C%9F%E6%97%B6%E9%97%B4/%E8%AE%A1%E6%97%B6%E5%99%A8%E5%B7%A5%E5%85%B7-TimeInterval)


## int转16进制字符串显示
```java
import cn.hutool.core.util.HexUtil;

// 结果一样
// 0x7460e8c0
// 0x7460e8c0
int code1 = "AaAaAa".hashCode();
int code2 = "BBAaBB".hashCode();
System.out.println(HexUtil.toHex(code1));
System.out.println(HexUtil.toHex(code2));
```

## 修改文件名
```java
import cn.hutool.core.io.FileUtil;
import cn.hutool.core.io.file.PathUtil;
import java.nio.file.Path;

// 只修改文件名, 不移动目录
PathUtil.rename(Path.of("/home/czz/test-1.png"), "new-file-001.png", false);

// 修改文件或目录的文件名，不变更路径，只是简单修改文件名
// e.g: FileUtil.rename(file, "aaa.jpg", false) xx/xx.png => xx/aaa.jpg

// param path       被修改的文件
// param newName    新的文件名，包括扩展名
// param isOverride 是否覆盖目标文件
// return 目标文件Path
// since 5.4.1
```

## 复制/移动文件
```java
// 根据url获取文件对象
java.io.File tmpFile = FileUtil.file("/home/czz/t-1.png");

// 复制文件到目录, 文件名不变
File targetFile = FileUtil.copy("/home/czz/test-1.png", "/home/czz/bak/", false);

// 移动文件到目录, 文件名不变, 也可以使用FileUtil.move(Path, Path, boolean)
java.nio.file.Path mPath = PathUtil.move(Path.of("/home/czz/test-1.png"), Path.of("/home/czz/bak/"), false);

// 移动文件或目录 当目标是目录时，会将源文件或文件夹整体移动至目标目录下 例如：
// move("/usr/aaa/abc.txt", "/usr/bbb")结果为："/usr/bbb/abc.txt"
// move("/usr/aaa", "/usr/bbb")结果为："/usr/bbb/aaa"
// 形参:
// src – 源文件或目录路径
// target – 目标路径，如果为目录，则移动到此目录下
// isOverride – 是否覆盖目标文件
// 返回值:
// 目标文件Path
// since: 5.5.1
```

## 文件路径及扩展名等
```java
String pathKey = "/xx-file/img/2023-11-11/xx-test.file.000.png";

// 返回文件名
// xx-test.file.000.png
String fileName = FileNameUtil.getName(pathKey);

// 获得文件的扩展名（后缀名），扩展名不带“.”
// png
String extName = FileNameUtil.extName(pathKey);

// 根据文件名检查文件类型，忽略大小写
// true
boolean result = FileNameUtil.isType(pathKey, "png", "jpg");
```

## 简单http服务器+上传文件
```java
package com.example.spring.cc;

import cn.hutool.core.net.multipart.UploadFile;
import cn.hutool.http.ContentType;
import cn.hutool.http.HttpUtil;

public class HttpMainTests {

    public static void main(String[] args) {
        // 简易Http服务器-SimpleServer
        HttpUtil.createServer(3333)
                // 设置http服务默认根目录
                .setRoot("/home/chenzz/web-html")
                // 文件上传
                .addAction("/upload-file", (request, response) -> {
                            // 接收通过form方式传过来的其他参数, 可以传多个参数
                            final String prompt = request.getParam("prompt");
                            // -------------------
                            final UploadFile file = request.getMultipart().getFile("file");
                            // 文件保存目录，默认读取HTTP头中的文件名进行保存
                            file.write("/home/chenzz/tempp/ttt/");
                            // 也可以使用自定义文件名保存
                            // file.write("/tmp/ttt/new_file.zip");
                            response.write("文件上传成功:" + file.getFileName(), ContentType.TEXT_PLAIN.toString());
                        }
                )
                .start();
    }

}
```


## 执行bash及命令
```java
import cn.hutool.core.util.RuntimeUtil;
import java.nio.charset.Charset;

// 在/home/czz目录执行ls命令
Process process = RuntimeUtil.exec(null, FileUtil.file("/home/czz"), "ls");
List<String> stringList = RuntimeUtil.getResultLines(process);
// -------------------------
for (String ss : stringList) {
    System.out.println(ss);
}

// 不指定目录直接执行shell, 返回执行结果
String str = RuntimeUtil.execForStr(Charset.forName("utf-8"), "adb logcat -c");
System.out.println(str);
```

## 时间格式化
```java
import cn.hutool.core.date.*;
import java.time.format.DateTimeFormatter;

DateTime now = DateTime.now();

DateUtil.format(now, DateTimeFormatter.ISO_DATE_TIME);
DateUtil.format(now, DateTimeFormatter.ISO_OFFSET_DATE_TIME);
DateUtil.format(now, DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ssxxx"));

// 2023-04-04T11:38:38.188+08:00[Asia/Shanghai]
// 2023-04-04T11:38:38.188+08:00
// 2023-04-04T11:38:38+08:00

// ------打印星期-------
// 星期二, now可以传Date日期
String weekText = DateUtil.dayOfWeekEnum(now).toChinese();
// 周二
now.dayOfWeekEnum().toChinese("周");
// 13:14 周二
DateUtil.format(now, "HH:mm EEE");


// 格式化日期间隔输出(毫秒间隔)
String betweenTimeText = DateUtil.formatBetween(15000 * 1000, BetweenFormatter.Level.MINUTE);
// 4小时10分
Console.log(betweenTimeText);
```

## 时间相关
```java
// 1. 农历日期，提供了生肖、天干地支、传统节日等方法
// https://hutool.cn/docs/index.html#/core/%E6%97%A5%E6%9C%9F%E6%97%B6%E9%97%B4/%E5%86%9C%E5%8E%86%E6%97%A5%E6%9C%9F-ChineseDate
ChineseDate date = new ChineseDate(DateUtil.parseDate("2023-11-11"));
// 癸卯年癸亥月癸酉日, 癸卯兔年 九月廿八
Console.log("{}, {}", date.getCyclicalYMD(), date);

// --------------------
// 2. DateTime及DateUtil中方法
DateTime time = DateTime.of("2024-02", "yyyy-MM");
// DateTime类实例方法
// 返回日期的最后一天, 29, 同DateUtil.getLastDayOfMonth
int lastDayOfMonth = time.getLastDayOfMonth();

// 获得年的部分
int year = time.year();

// 返回月份(从0开始), 1
// 同DateUtil.month
int month = time.month();

// 返回季度(从1开始), 1
// 同DateUtil.quarter
int quarter = time.quarter();

// 指定日期是所在年份的第几周
int weekOfYear = time.weekOfYear();

// 指定日期是所在月份的第几周
int weekOfMonth = time.weekOfMonth();

// 指定日期是这个日期所在年份的第几天，从1开始
int dayOfYear = time.dayOfYear();

// 指定日期是这个日期所在月份的第几天，从1开始
int dayOfMonth = time.dayOfMonth();
// 针对以上方法, 在DateUtil也有同名的对应方法
```

## 返回从指定日期开始的n天时间
```java
BiFunction<DateTime, Integer, List<String>> funWeek = (weekM1, count) -> Stream.iterate(0, nn -> nn + 1).limit(count)
        .map(oo -> {
            DateTime ooNewTime = weekM1.offsetNew(DateField.DAY_OF_YEAR, oo);
            return DateUtil.format(ooNewTime, "MM-dd");
        })
        .toList();

// 周一
DateTime m1 = DateTime.of("2023-11-27", "yyyy-MM-dd");
var timeList = funWeek.apply(m1, 5);
// ["11-27","11-28","11-29","11-30","12-01"]
```

## 根据起始日期显示季度月数据
```java
// 结束日期
DateTime endTime = DateTime.of("2024-05-09", "yyyy-MM-dd");
// 季度开始日期
DateTime beginTime = DateUtil.beginOfQuarter(endTime);

// 两个日期相差的月份数
int monthCount = (int) DateUtil.betweenMonth(beginTime, endTime, true);
Console.log("两个日期相差的月份数量: {}", monthCount);
// tip: 如果起始日期为同一个月, monthCount结果为0, 以下会执行从0开始的循环, 并遍历1次后结束
IntStream.rangeClosed(0, monthCount).boxed()
        // 从大到小排序
        .sorted((o1, o2) -> Integer.compare(o2, o1))
        .peek(System.out::println)
        .map(n -> {
            DateTime nTime = beginTime.offsetNew(DateField.MONTH, n);
            return Dict.create().set("index", n)
                    .set("month", nTime.month())
                    .set("monthEngText", nTime.monthEnum())
                    .set("time", nTime);
        })
        .forEach(System.out::println);

// {index=1, month=4, monthEngText=MAY, time=2024-05-01 00:00:00}
// {index=0, month=3, monthEngText=APRIL, time=2024-04-01 00:00:00}
```


## 计算指定xx年第n周日期等数据
```java
// =================================
BiFunction<DateTime, Integer, Dict> fun = (yearBegin, nn) -> {
    // yearBegin, 当年开始时间
    // nn, 第几周
    Dict nnDict = Dict.create();

    // --------------------------
    DateTime nnTime = yearBegin.offsetNew(DateField.WEEK_OF_YEAR, nn - 1);
    DateTime nnTimeWeekStart = DateUtil.beginOfWeek(nnTime);
    DateTime nnTimeWeekEnd = DateUtil.endOfWeek(nnTime);

    int nnYear = yearBegin.getField(DateField.YEAR);
    String nnM1 = DateUtil.format(nnTimeWeekStart, "MM/dd");
    String nnM7 = DateUtil.format(nnTimeWeekEnd, "MM/dd");
    // --------------------------

    // 2023年第50周, 12/04至12/10
    String nnText = StrUtil.format("{}年第{}周, {}至{}", nnYear, nn, nnM1, nnM7);
    // 是否当前周
    boolean currentWeek = DateUtil.weekOfYear(DateTime.now()) == nn;
    nnDict.set("year", nnYear);
    nnDict.set("weekIndex", nn);
    nnDict.set("m1", nnM1);
    nnDict.set("m7", nnM7);
    nnDict.set("currentWeek", currentWeek);
    nnDict.set("text", nnText);
    return nnDict;
};

DateTime now = DateTime.now();
int xx = DateUtil.weekOfYear(now);
log.info("当前时间在本年是第{}周", xx);

// 获取今年的第n周
DateTime thisYearBegin = DateTime.of("2023-01-01", "yyyy-MM-dd");
// DateTime thisYearBegin = DateUtil.beginOfYear(now);
log.info("今年开始时间:{}", thisYearBegin);

// =================================
// 今年第n周
Dict ffDict = fun.apply(thisYearBegin, 51);
System.out.println(ffDict);

// 生成最近前2周后两周数据
List<Dict> dictList = new ArrayList<>();
for (int i = xx - 2; i < xx + 2; i++) {
    dictList.add(fun.apply(thisYearBegin, i));
}

log.info("最近周数据: {}", JSONUtil.toJsonPrettyStr(dictList));
```

## List列表转数组
```java
import cn.hutool.core.util.ArrayUtil;

List<String> fileNames = CollUtil.newArrayList("/home/czz/f1.txt", "/usr/local/java", "/root/logs/f1.log");
// 1.使用ArrayUtil工具类实现
String[] fileArr = ArrayUtil.toArray(fileNames, String.class);

// 2.使用toArray实现, Integer[]::new, Object[]::new
// 数组引用, 语法格式为: Type[]::new
String[] fileStrArr1 = fileNames.stream().toArray(String[]::new);
String[] fileStrArr2 = fileNames.toArray(String[]::new);
// 不传参则转换成Object数组
Object[] fileStrArr3 = fileNames.toArray();
```

## 数组转List
```java
import cn.hutool.core.collection.CollUtil;

String[] strArr = {"java8", "is", "easy", "to", "use"};

List<String> stringList1 = CollUtil.newArrayList(strArr);
List<String> stringList2 = CollUtil.newLinkedList(strArr);
// Returns an unmodifiable list containing an arbitrary number of elements. See Unmodifiable Lists for
List<String> unmodifiableList = List.of(strArr);
```

## 数操作NumberUtil
```java
import cn.hutool.core.util.NumberUtil;

// 计算和(如果为null当作0)
// 565.68
BigDecimal total = NumberUtil.add(BigDecimal.valueOf(555.68), BigDecimal.TEN, BigDecimal.ZERO, null, null);

// compare比较, add相加, sub减, mul乘, div除, 进制转换, 四舍五入等
```

## 正则表达式使用
```java
import cn.hutool.core.lang.PatternPool;
import cn.hutool.core.util.ReUtil;

// 提取出字符串中的单词: A,C,happy
List<String> matchStr1 = ReUtil.findAllGroup0(PatternPool.WORD, "选项A: 维生素C是一种人体必需品, happy!");
Console.log("{}", JSONUtil.toJsonPrettyStr(matchStr1));

// 提取出字符串中的数字:2023,9,3
List<String> matchStr2 = ReUtil.findAllGroup0(PatternPool.NUMBERS, "张三在2023年由9个班级参加的比赛中，获得了第3名！");
Console.log("{}", JSONUtil.toJsonPrettyStr(matchStr2));

//\\s匹配空格符、制表符、换行符和回车符
// 去除字符串中所有空格: java高级教程springboot3.x-web篇-01.avi
String newStr = ReUtil.replaceAll("  java高级教程 spring boot 3.x    -web篇-01.avi  ", "\\s", "");
Console.log(newStr);

// --------------------------------
// 使用String的replaceAll实现文件名处理
String fileNameWithIllegalChars = "/path/to/my！:file!name*?.txt!";

String illegalChars = "[\\/:\"*?<>|!！]";
// 去除非法文件名字符
String validFileName = fileNameWithIllegalChars.replaceAll(illegalChars, "");
// 去除空格字符
validFileName = validFileName.replaceAll("\\s", "");
// 结果: pathtomyfilename.txt
System.out.println(validFileName);
```

---------------------
- [hutool 文档](https://hutool.cn/docs/index.html)
