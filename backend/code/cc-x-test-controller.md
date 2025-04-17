## hutool db操作数据库返回数据

```java
package com.cc.cy.api.controller;

import cn.hutool.core.date.DateUtil;
import cn.hutool.core.lang.Console;
import cn.hutool.core.lang.Dict;
import cn.hutool.core.text.StrBuilder;
import cn.hutool.core.util.ArrayUtil;
import cn.hutool.core.util.StrUtil;
import cn.hutool.db.Db;
import cn.hutool.db.Entity;
import cn.hutool.db.Page;
import cn.hutool.db.PageResult;
import cn.hutool.db.sql.Condition;
import cn.hutool.json.JSONArray;
import cn.hutool.json.JSONObject;
import cn.hutool.json.JSONUtil;
import com.cc.common.core.controller.BaseController;
import com.cc.common.core.domain.AjaxResult;
import com.cc.common.core.domain.cy.HtSqlCondition;
import com.cc.common.utils.spring.SpringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.sql.DataSource;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

/**
 * hutool db test
 *
 * @author chenzz
 */
@RestController
@RequestMapping("/cc/x-test")
public class CcTestController extends BaseController {

    private static final Logger log = LoggerFactory.getLogger(CcTestController.class);

    /**
     * 用户登录日志分页
     */
    @GetMapping("/user-page")
    public AjaxResult xxPage(@RequestParam(name = "pageNum", defaultValue = "1") Integer pageNum, @RequestParam(name = "pageSize", defaultValue = "15") Integer pageSize) {
        DataSource ds = SpringUtils.getBean("masterDataSource");
        Db db = Db.use(ds);
        try {
            PageResult<Entity> result = db.page("select * from sys_logininfor order by info_id desc", new Page(pageNum - 1, pageSize));

            if (result.getTotal() > 0) {
                var listUserName = result.stream().map(tt -> tt.getStr("user_name")).distinct().toList();
                List<Entity> userEntityList = db.findBy("sys_user",
                        new Condition("user_name", "in", String.join(",", listUserName))
                );
                // List<Entity> userRowList = db.query("select * from sys_user order by user_id desc limit 100");

                List<Dict> dataList = result.stream().map(tt -> {
                            Dict xxDict = Dict.create()
                                    .set("id", tt.getLong("info_id"))
                                    .set("登录用户(手机号)", tt.getStr("user_name"));

                            Optional<Entity> optionalEntity = userEntityList.stream().filter(yy -> yy.getStr("user_name").equals(tt.getStr("user_name"))).findAny();

                            if (optionalEntity.isPresent()) {
                                Entity ooEntity = optionalEntity.get();
                                xxDict.set("用户姓名", ooEntity.getStr("nick_name"));
                            } else {
                                xxDict.set("用户姓名", "");
                            }
                            xxDict.set("登录ip", tt.getStr("ipaddr"));
                            xxDict.set("登录位置", tt.getStr("login_location"));
                            xxDict.set("浏览器", tt.getStr("browser"));
                            xxDict.set("操作系统", tt.getStr("os"));
                            xxDict.set("结果", tt.getStr("msg"));
                            xxDict.set("登录时间", DateUtil.formatDateTime(tt.getDate("login_time")));
                            return xxDict;
                        }
                ).toList();

                Dict dict = Dict.create()
                        .set("total", result.getTotal())
                        .set("current", pageNum)
                        .set("size", pageSize)
                        .set("pages", result.getTotalPage())
                        .set("records", dataList);
                return success(dict);
            }

        } catch (Exception ex) {
            log.error("error: {}", ex.getMessage());
        }
        return error();
    }

    /**
     * 知识文档分页
     * 多条件and复合查询
     */
    @GetMapping("/doc-page")
    public AjaxResult docPage(@RequestParam(name = "pageNum", defaultValue = "1") Integer pageNum, @RequestParam(name = "pageSize", defaultValue = "15") Integer pageSize, Integer type, String tag, String title) throws SQLException {
        DataSource ds = SpringUtils.getBean("masterDataSource");
        Db db = Db.use(ds);

        // ------------------------------
        // ------------------------------
        List<HtSqlCondition> whereList = new ArrayList<>();
        if (StrUtil.isNotEmpty(tag)) {
            // json字段匹配, eg.{"tags": ["CPU", "国产", "申威", "指令集"], "type": 99}
            whereList.add(new HtSqlCondition("JSON_CONTAINS(config_json,JSON_OBJECT('tags', ?))", tag));
        }
        if (StrUtil.isNotEmpty(title)) {
            // like模糊搜索
            whereList.add(new HtSqlCondition("title like ?", "%" + title + "%"));
        }
        if (type != null) {
            whereList.add(new HtSqlCondition("type=?", type));
        }

        var sqlBuilder = StrBuilder.create("select * from cc_document");
        if (!whereList.isEmpty()) {
            sqlBuilder.append(" where ")
                    .append(whereList.stream().map(HtSqlCondition::getCondition).collect(Collectors.joining(" and ")));
        }
        sqlBuilder.append(" order by create_time desc");
        Console.log(sqlBuilder);


        var hPage = new cn.hutool.db.Page(pageNum - 1, pageSize);
        PageResult<Entity> result;
        if (whereList.isEmpty()) {
            result = db.page(sqlBuilder.toString(), hPage);
        } else {
            // 这里需要先把List转换成数组
            Object[] valueObjArr = ArrayUtil.toArray(whereList.stream().map(HtSqlCondition::getValue).toList(), Object.class);
            // 使用Stream/List的toArray生成数组
            // 数组引用, 语法格式为: Type[]::new
            // HtSqlCondition[] conditionArr = whereList.toArray(HtSqlCondition[]::new);
            // Object[] valueObjArr = whereList.stream().map(HtSqlCondition::getValue).toArray(Object[]::new);
            // Object... params可变参数必须传数组, 不能传List(条件查询不起作用)
            result = db.page(sqlBuilder.toString(), hPage, valueObjArr);
        }
        // ------------------------------
        // ------------------------------

        if (result.getTotal() > 0) {
            List<Dict> dataList = result.stream().map(tt -> {
                        Dict dict = Dict.create();

                        dict.set("id", tt.getStr("id"));
                        dict.set("fileExtension", tt.getStr("file_extension"));
                        dict.set("fileMd5", tt.getStr("file_md5"));
                        dict.set("fileSize", tt.getLong("file_size"));
                        dict.set("url", tt.getStr("url"));
                        dict.set("originalFileName", tt.getStr("original_file_name"));
                        dict.set("title", tt.getStr("title"));
                        dict.set("createTime", DateUtil.formatDateTime(tt.getDate("create_time")));

                        String configJson = tt.getStr("config_json");
                        if (JSONUtil.isTypeJSONObject(configJson)) {
                            JSONObject jsonObjectConfig = JSONUtil.parseObj(configJson);
                            JSONArray tagsArray = jsonObjectConfig.getJSONArray("tags");
                            dict.set("tags", tagsArray == null ? Collections.emptyList() : tagsArray);
                        } else {
                            dict.set("tags", Collections.emptyList());
                        }

                        return dict;
                    }
            ).toList();

            var resultDict = Dict.create()
                    .set("total", result.getTotal())
                    .set("current", pageNum)
                    .set("size", pageSize)
                    .set("pages", result.getTotalPage())
                    .set("records", dataList);
            return success(resultDict);
        }

        // 返回空数据
        return success(Dict.create()
                .set("total", 0)
                .set("current", pageNum)
                .set("size", pageSize)
                .set("pages", 0)
                .set("records", Collections.emptyList()));
    }

}
```

## HtSqlCondition
```java
package com.cc.common.core.domain.cy;

import lombok.Getter;
import lombok.Setter;

/**
 * hutool自定义查询条件构造对象
 * 目前仅支持and多条件
 *
 * @author chenzz
 */
@Setter
@Getter
public class HtSqlCondition {

    /**
     * 条件
     */
    private String condition;

    /**
     * 值
     */
    private Object value;

    public HtSqlCondition(String condition, Object value) {
        this.condition = condition;
        this.value = value;
    }
}
```