> mybatis-plus-boot-starter版本>=3.5.7

> package
```java
import cn.hutool.core.collection.CollUtil;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.LambdaUpdateWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.core.toolkit.Wrappers;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
```

## 添加数据
```java
SysUserEntity entity = SysUserEntity.builder().loginName("czz").email("czz@qq.com").phonenumber("13800138000").delFlag(0).build();
// 添加数据并返回自增id
boolean result = userService.save(entity);
```


## 条件查询, 删除
```java
// --------1.LambdaQueryWrapper---------
// 查询
LambdaQueryWrapper<SysUserEntity> queryWrapper = new LambdaQueryWrapper<>();
// 设置查询字段, 可以针对大字段进行过滤
// queryWrapper.select(SysUserEntity::getUserId, SysUserEntity::getLoginName, SysUserEntity::getAvatar, SysUserEntity::getPhonenumber);
queryWrapper.eq(SysUserEntity::getLoginName, "chenzz")
        .gt(SysUserEntity::getUserId, 100);

// -----------------
// 使用apply方法拼接 sql
// 例1: apply("id = 1")
// 例2: apply("date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")
// 例3: apply("date_format(dateColumn,'%Y-%m-%d') = {0}", LocalDate.now())
// 例4: apply("name={0,javaType=int,jdbcType=NUMERIC,typeHandler=xxx.xxx.MyTypeHandler}", "老王")
// 搜索json字段, 搜索指定区域的管理员
queryWrapper.apply("JSON_CONTAINS(config_json,JSON_OBJECT('region_ids', {0}))", 360700)
// -----------------

// 查询时排序, 分组(删除可以不需要)
// 多字段排序, 优先级高的放前面,优先级低在后面
queryWrapper.orderByAsc(SysUserEntity::getUserType, SysUserEntity::getStatus)
        .orderByDesc(SysUserEntity::getCreateTime)
        .groupBy(SysUserEntity::getUserType);

List<SysUserEntity> list = userMapper.selectList(queryWrapper);
// 分页查询, 页码从1开始
IPage<SysUserEntity> pageList = userMapper.selectPage(new Page<>(1, 15), queryWrapper);

// 条件删除
// int result = userMapper.delete(queryWrapper);

// --------2.使用Wrappers实现---------
List<SysUserEntity> list2 = userMapper.selectList(Wrappers.<SysUserEntity>lambdaQuery().orderByDes(SysUserEntity::getCreateTime));
int deleteRows = userMapper.delete(Wrappers.<SysUserEntity>lambdaQuery().in(SysUserEntity::getUserId, CollUtil.newLinkedList(1, 3, 5)));
```


## 更新数据
```java
// 1.LambdaUpdateWrapper
LambdaUpdateWrapper<SysUserEntity> updateWrapper = new LambdaUpdateWrapper<>();
updateWrapper.set(SysUserEntity::getLoginName, "chenzz")
        .set(SysUserEntity::getEmail, "chenzz@qq.com")
        .set(SysUserEntity::getPhonenumber, "13800138000")
        // 清空字段
        .set(SysUserEntity::getLoginDate, null)
        // ---------where----------
        .eq(SysUserEntity::getUserId, 100);

int rows = userMapper.update(updateWrapper);

// 2.使用Wrappers实现
int result = userMapper.update(Wrappers.<SysUserEntity>lambdaUpdate().set(SysUserEntity::getEmail, "czz@qq.com").eq(SysUserEntity::getUserId, 100));

// ----------3----------
// 更新json字段数据
// JSON_SET, 用于设置或更新JSON对象中的键值对5。如果键不存在，则添加新键值对；如果键已存在，则更新其值5。
// UPDATE test SET data = JSON_SET(data, '$.age', 31) WHERE id = 1;

// JSON_REPLACE, 仅当键已存在时替换其值，不会添加新键5。 示例： 1
// UPDATE test SET data = JSON_REPLACE(data, '$.age', 32) WHERE id = 1;

// JSON_REMOVE, 用于删除JSON对象中的指定键值
// UPDATE test SET data = JSON_REMOVE(data, '$.age') WHERE id = 1;

// 根据id更新json字段
// sql: UPDATE sys_user SET config_json = JSON_SET(config_json, '$.accountType', 99) WHERE deleted = 0 AND (id = ?)
int updateRows = userMapper.update(Wrappers.<SysUserEntity>lambdaUpdate()
    .setSql(cn.hutool.core.util.StrUtil.format("config_json = JSON_SET(config_json, '$.accountType', {})", 99))
    .eq(SysUserEntity::getUserId, 1L)
);
```
----------------------------------


## 整体示例
```java
package com.example.demo;

import cn.hutool.core.collection.CollUtil;
import cn.hutool.core.date.DateUtil;
import cn.hutool.core.lang.Dict;
import cn.hutool.json.JSONUtil;
import com.baomidou.mybatisplus.core.conditions.Wrapper;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.LambdaUpdateWrapper;
import com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.core.toolkit.CollectionUtils;
import com.baomidou.mybatisplus.core.toolkit.Wrappers;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.example.demo.framework.entity.R;
import com.example.demo.project.mpsample.entity.SysUserEntity;
import com.example.demo.project.mpsample.mapper.SysUserMapper;
import com.example.demo.project.mpsample.service.ISysUserService;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.ContextConfiguration;

import javax.annotation.Resource;
import java.util.Date;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

import static org.assertj.core.api.Assertions.assertThat;

@DisplayName("MyBatis Plus3.x测试")
@SpringBootTest
@ContextConfiguration(classes = DemoApplication.class)
public class MyBatisPlus3xTests {

    //------------------------------

    @Autowired
    private ApplicationContext context;

    @Autowired
    private ISysUserService userService;

    @Resource
    private SysUserMapper userMapper;

    /**
     * 使用QueryWrapper查询数据列表
     */
    @Test
    void testSelectWrappers() {
        // DataSource ds = context.getBean("dataSource", DataSource.class);
        // 查询email为qq.com结尾的root用户
        Wrapper<SysUserEntity> wrapper = new QueryWrapper<SysUserEntity>().lambda()
                .likeLeft(SysUserEntity::getEmail, "@qq.com")
                .eq(SysUserEntity::getLoginName, "root")
                .orderByAsc(SysUserEntity::getCreateTime)
                .groupBy(SysUserEntity::getUserType);

        //------------------------
        List<SysUserEntity> list = userService.list(wrapper);
        List<SysUserEntity> list2 = userMapper.selectList(wrapper);
        printList(list);
        // ====================================
        // 分页查询, 页码从1开始
        IPage<SysUserEntity> page1 = userService.page(new Page<>(1, 20), wrapper);
        IPage<SysUserEntity> page2 = userMapper.selectPage(new Page<>(1, 15), wrapper);
        System.out.println("wrapper sql: " + wrapper.getSqlSegment());
        System.out.println("target sql: " + wrapper.getTargetSql());
        printList(page2.getRecords());
        // 添加断言
        // --------------------------
        assertThat(list2).isNotNull();
        boolean booleanAssert = page1.getTotal() > -1;
        assertThat(booleanAssert).isTrue();
    }

    /**
     * 使用LambdaUpdateWrapper, UpdateWrapper<>().lambda()更新记录
     */
    @Test
    void testUpdateWrapper() {
        // 更新记录
        // Wrapper<SysUserEntity> wrapperUpdate = Wrappers.<SysUserEntity>lambdaUpdate().set(SysUserEntity::getLoginName, "chenzz").eq(SysUserEntity::getUserId, 100)
        Wrapper<SysUserEntity> wrapper = new UpdateWrapper<SysUserEntity>().lambda()
                // -------------------
                .set(SysUserEntity::getLoginName, "chenzz")
                .set(SysUserEntity::getEmail, "chenzz@qq.com")
                .set(SysUserEntity::getPhonenumber, "13800138000")
                // 清空登录时间字段
                .set(SysUserEntity::getLoginDate, null)
                // -------------------
                .eq(SysUserEntity::getUserId, 100);

        boolean result = userService.update(wrapper);
        System.out.println("update result=" + result);
        // =========================================
        int rows = userMapper.update(null, wrapper);
        System.out.println(rows + " rows affected");
        // =========================================
        // 所有更新成功标志(使用&&运算符)
        boolean allUpdateOk = result && (rows > 0);
        System.out.println("allUpdateOk=" + allUpdateOk);
        assertThat(allUpdateOk).isTrue();

        // ===============================
        LambdaUpdateWrapper<SysUserEntity> updateWrapper = new LambdaUpdateWrapper<>();
        updateWrapper.set(SysUserEntity::getLoginName, "tim")
                .set(SysUserEntity::getEmail, "tim@qq.com");
        // -------------------
        updateWrapper.eq(SysUserEntity::getUserId, 200);
        System.out.println("------sql where------\r\n" + updateWrapper.getTargetSql());
    }

    /**
     * 测试Wrapper(select/update)
     * Lambda
     */
    @Test
    void testWrapper() {
        // 查询(delete删除也可以用此类wrapper)
        // SELECT * FROM sys_user WHERE del_flag=0 AND (login_name = ? AND user_id > ?) GROUP BY sex ORDER BY user_type ASC,status ASC,create_time DESC
        LambdaQueryWrapper<SysUserEntity> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(SysUserEntity::getLoginName, "chenzz")
                .gt(SysUserEntity::getUserId, 100);
        // 多字段排序, 优先级高的放前面,优先级低的在后面
        queryWrapper.orderByAsc(SysUserEntity::getUserType, SysUserEntity::getStatus)
                .orderByDesc(SysUserEntity::getCreateTime)
                .groupBy(SysUserEntity::getSex);

        long rowsCount = userMapper.selectCount(queryWrapper);
        System.out.println("rows=" + rowsCount);

        List<SysUserEntity> list = userMapper.selectList(queryWrapper);
        printList(list);
        // ------------------------------------------
        // 更新
        LambdaUpdateWrapper<SysUserEntity> updateWrapper = new LambdaUpdateWrapper<>();
        updateWrapper.set(SysUserEntity::getLoginName, "chenzz")
                .set(SysUserEntity::getEmail, "chenzz@qq.com")
                .set(SysUserEntity::getPhonenumber, "13800138000")
                // -------------------
                .eq(SysUserEntity::getUserId, 100);

        int rows = userMapper.update(null, updateWrapper);
        System.out.println(rows + " rows affected");
        assertThat(rows).isGreaterThan(-1);
    }

    /**
     * 测试复杂查询Wrapper
     */
    @Test
    void testAndOrWrapper() {
        // Execute SQL：SELECT user_id,login_name,user_name,user_type,email,phonenumber,sex,avatar,password,salt,status,del_flag,login_ip,login_date,create_time FROM sys_user
        // WHERE del_flag=0 AND (user_id <> '1' AND ((user_name = 'chenzz' AND user_type = '00') OR (phonenumber = '13888888888' AND status = 0)))
        // GROUP BY user_type ORDER BY create_time ASC
        // 查询
        LambdaQueryWrapper<SysUserEntity> queryWrapper = new LambdaQueryWrapper<>();
        // queryWrapper.select(SysUserEntity::getUserId, SysUserEntity::getLoginName, SysUserEntity::getAvatar, SysUserEntity::getPhonenumber)
        // AND a.user_id <> 1
        queryWrapper.ne(SysUserEntity::getUserId, "1");
        // AND ((a.`user_name` = 'chenzz' AND a.user_type = '00') OR (a.phonenumber = '13888888888' AND a.status = 0))
        queryWrapper.and(oo -> (oo.and(tt -> tt.eq(SysUserEntity::getUserName, "chenzz").eq(SysUserEntity::getUserType, "00")))
                .or(pp -> pp.eq(SysUserEntity::getPhonenumber, "13888888888").eq(SysUserEntity::getStatus, 0)));
        queryWrapper.orderByAsc(SysUserEntity::getCreateTime)
                .groupBy(SysUserEntity::getUserType);

        System.out.println("------sql where------\r\n" + queryWrapper.getTargetSql());
        // 查询结果
        List<SysUserEntity> list = userMapper.selectList(queryWrapper);

        int rowsCount = list.size();
        System.out.println("rows=" + rowsCount);
        assertThat(rowsCount).isEqualTo(0);
    }

    /**
     * 使用QueryWrapper根据条件删除记录
     */
    @Test
    void testDeleteQueryWrapper() {
        // Wrappers.<?>lambdaQuery()等同于new QueryWrapper<?>().lambda()
        // Wrappers.<SysUserEntity>lambdaQuery().eq(SysUserEntity::getEmail, "chen@qq.com")
        Wrapper<SysUserEntity> wrapper = Wrappers.<SysUserEntity>lambdaQuery()
                .like(SysUserEntity::getLoginName, "ooOoo")
                .ge(SysUserEntity::getUserId, 100);

        boolean result = userService.remove(wrapper);
        System.out.println("delete result=" + result);
        // =========================================
        int rows = userMapper.delete(wrapper);
        System.out.println(rows + " rows affected");
        // =========================================
        int deleteRows = userService.getBaseMapper().delete(Wrappers.<SysUserEntity>lambdaQuery().in(SysUserEntity::getUserId, CollUtil.newLinkedList(1, 3, 5)));
        System.out.println(deleteRows + " rows delete");
        assertThat(deleteRows).isEqualTo(3);
    }

    /**
     * 其他crud操作
     */
    @Test
    void testCrud() {
        System.err.println("删除一条数据：" + userService.removeById(9L));
        // int rows = userMapper.deleteById(9L);

        SysUserEntity entity = SysUserEntity.builder().loginName("czz").email("czz@qq.com").phonenumber("13800138000").delFlag(0).build();
        // 添加数据并返回自增id
        boolean result = userService.save(entity);
        // int rows2 = userMapper.insert(entity);
        // 自动回写的ID,用户ID,自增
        Long id = entity.getUserId();
        System.out.println("insert数据的cid=" + id);
        System.err.println("插入的数据：" + result + ", 插入信息：" + entity.toString());
        // ----------------------------------------------------------------
        System.err.println("根据id主键查询：" + userService.getById(id).toString());
        // userMapper.selectById(id);
        System.err.println("查询单个：" +
                userMapper.selectOne(Wrappers.<SysUserEntity>lambdaQuery().eq(SysUserEntity::getLoginName, "czz"))
                        .toString());
        assertThat(id > 0).isTrue();
    }

    @DisplayName("数据查询等操作测试")
    @Test
    void testOoo() {
        printList(userMapper.selectList(null));

        // int result = userMapper.delete(Wrappers.<SysUserEntity>lambdaQuery().ge(SysUserEntity::getUserId, 0));
        // System.out.println(result);
        assertThat("select test").isNotNull();
    }

    @DisplayName("插入时间测试")
    @Test
    void testOoo2() {
        // 插入的时间使用java程序进行处理, 可以保证返回的时候和预期一致
        Date beginOfDay = DateUtil.beginOfDay(DateUtil.parse("2021-11-11", "yyyy-MM-dd"));
        System.out.println(beginOfDay);

        SysUserEntity entity = SysUserEntity.builder().loginName("czz").email("czz@qq.com").phonenumber("13800138000").delFlag(0).loginDate(beginOfDay).build();
        boolean result = userService.save(entity);
        System.out.println(result);
        // ----------------------------------------------------------------
        SysUserEntity newEntity = userMapper.selectById(entity.getUserId());
        System.err.println("时间：" + DateUtil.formatDateTime(newEntity.getLoginDate()));

        assertThat(newEntity.getLoginDate().equals(beginOfDay)).isTrue();
    }


    /**
     * 对象转换为json文本
     */
    @DisplayName("对象转换为json文本")
    @Test
    void testR2JsonText() {
        List<SysUserEntity> list = userMapper.selectList(null);
        R result = R.ok().data(list);
        // -------------------------
        // JSONUtil->parseXXX和toXXX, 这两种方法主要是针对JSON和其它对象之间的转换
        // List<Dict> listDict = JSONUtil.toList(json, Dict.class);
        // String jsonText = JSONUtil.toJsonStr(result);
        // 格式化后的JSON字符串
        String prettyJson = JSONUtil.toJsonPrettyStr(result);
        System.out.println("-----------------------------");
        System.out.println("1. 格式化后json:\r\n" + prettyJson);
        // --------------------------
        // 2. 格式化后json(null值不显示):
        // {
        //     "msg": "ok",
        //     "code": 0
        // }
        Dict dict = Dict.create().set("code", 0).set("msg", "ok").set("data", null);
        String jsonText = JSONUtil.parse(dict).toString();
        System.out.println("2. 未格式化json:\r\n" + jsonText);

        assertThat(dict).isNotEmpty();
    }

    //======================================
    private void printListMap(List<Map<String, Object>> list) {
        list.stream().forEach(mm -> {
            System.out.println("============================");
            // map遍历-1
            mm.entrySet().forEach(t -> System.out.println(t.getKey() + ": " + t.getValue()));
            // map遍历-2
            mm.forEach((k, v) -> {
                System.out.println(k + ": " + v);
            });
        });
    }

    private <T> void printList(List<T> list) {
        if (!CollectionUtils.isEmpty(list)) {
            list.forEach(System.out::println);
        }
    }

    @DisplayName("Map的遍历")
    @Test
    void testMap() {
        Map<String, String> map = new LinkedHashMap<>();
        map.put("熊大", "棕色");
        map.put("熊二", "黄色");
        // key
        for (String key : map.keySet()) {
            System.out.println(key);
        }
        // value
        for (String value : map.values()) {
            System.out.println(value);
        }

        assertThat(map).isNotNull();
    }

}
```

---------------------
- [MyBatis Plus文档](https://baomidou.com/)