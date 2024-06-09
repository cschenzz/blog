## Function函数实现集合排序和映射
```java
import java.util.*;
import java.util.function.Function;

import cn.hutool.db.Db;
import cn.hutool.db.Entity;
import cn.hutool.core.lang.Dict;
import cn.hutool.core.date.DateUtil;
import cn.hutool.core.collection.CollUtil;
import cn.hutool.json.JSONUtil;

Function<Collection<Entity>, List<Dict>> funGenList = (xxEntityList) -> xxEntityList.stream()
        .sorted((o1, o2) -> {
                    // Date x1 = o1.getDate("上门回访时间");
                    // Date x2 = o2.getDate("上门回访时间");
                    // 最近回访的排前面
                    // return x2.compareTo(x1);

                    // ==========================
                    // 先根据星期升序, 再根据客户经理升序排列
                    int x1 = o1.getInt("weekNum");
                    int x2 = o2.getInt("weekNum");

                    String y1 = o1.getStr("客户经理");
                    String y2 = o2.getStr("客户经理");

                    int sss = Integer.compare(x1, x2);
                    return sss == 0 ? y1.compareTo(y2) : sss;
                }
        )
        .map(tt -> {
                    Date tempLastVisitTime = tt.getDate("上门回访时间");
                    Dict dict = Dict.create()
                            .set("公司名", tt.getStr("公司名"))
                            .set("客户经理", tt.getStr("客户经理"));
                    dict.set("合同到期日期", DateUtil.formatDate(tt.getDate("合同到期日期")));
                    dict.set("回访时间", DateUtil.formatDate(tempLastVisitTime));
                    dict.set("最后回访离现在天数", DateUtil.betweenDay(tempLastVisitTime, DateTime.now(), false));
                    return dict;
                }
        )
        .toList();

// ===============================
Db db = Db.use(getDs());
List<Entity> entityList = db.query("select * from yy_客户拜访记录 limit 10");
List<Dict> dictList = funGenList.apply(entityList);
System.out.println(JSONUtil.toJsonPrettyStr(dictList));
```

## 使用Function校验list列表参数字段
```java
/**
 * 校验函数, 填写了客户必须填写预约时间, 并需要对预约时间进行校验
 * 返回错误信息; 空表示没有错误
 * ==============参数校验==============
 */
Function<Collection<AddDailyPaperDto.VisitCustomer>, String> funErrCheck = (list) -> {
    // 预约时间参数校验, 返回错误信息; 空表示没有错误
    for (var oo : list) {
        if (StrUtil.isNotEmpty(oo.getCustomerName())) {
            if (StrUtil.isEmpty(oo.getExpectVisitTime())) {
                return "请填写客户[" + oo.getCustomerName() + "]的预约拜访时间!";
            }
            DateTime expectVisitTime = DateUtil.parseUTC(oo.getExpectVisitTime());
            if (expectVisitTime.before(DateTime.now())) {
                return "客户[" + oo.getCustomerName() + "]预约拜访时间不能是过去的时间!";
            }
            if (expectVisitTime.after(DateTime.now().offset(DateField.DAY_OF_MONTH, 7))) {
                return "客户[" + oo.getCustomerName() + "]预约拜访时间不能太久!";
            }
        }
    }
    return "";
};

// 在Controller中使用并返回错误信息
String errMsg1 = funErrCheck.apply(dto.getCustomerInService());
if (StrUtil.isNotEmpty(errMsg1)) {
    return error(errMsg1);
}
```

## 使用forEach处理集合中数据
```java
List<Dict> dictList = CollUtil.newArrayList(
        Dict.create().set("email", "czz@qq.com").set("loginTime", DateTime.of("2023-07-11 08:00:00", "yyyy-MM-dd HH:mm:ss")),
        Dict.create().set("email", "ym@qq.com").set("loginTime", DateTime.of("2023-07-14 09:08:00", "yyyy-MM-dd HH:mm:ss")),
        Dict.create().set("email", "cyh@163.com").set("loginTime", DateTime.of("2023-07-15 18:20:00", "yyyy-MM-dd HH:mm:ss"))
);

log.debug("{}", JSONUtil.toJsonPrettyStr(dictList));
// 使用forEach处理集合中数据(添加或者修改字段等)
dictList.forEach(oo -> {
    Date tempLoginTime = oo.getDate("loginTime");
    long days = DateUtil.between(tempLoginTime, DateTime.now(), DateUnit.DAY);

    oo.set("登录时间", DateUtil.formatDateTime(tempLoginTime));
    oo.set("距离现在天数", days);
    oo.set("days", days);
});

log.debug("forEach处理数据:{}", JSONUtil.toJsonPrettyStr(dictList));

// list先过滤再遍历
dictList.stream().filter(oo -> oo.getLong("days") > 30).forEach(oo -> {
    log.info("1.--{}", oo);
    // 可以进行其他数据处理
});
```