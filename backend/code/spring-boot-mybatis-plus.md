spring boot + mybatis plus 数据库crud示例
实现restful接口, 包括分页, 列表, 添加, 修改, 删除数据

> mybatis-plus-boot-starter版本>=3.5.7

> jdk17已用jakarta替代javax

## controller
```java
package com.example.demo.api.controller;

import cn.hutool.core.collection.CollUtil;
import cn.hutool.core.date.DateUtil;
import cn.hutool.core.lang.Dict;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.example.demo.api.dto.AddSysUserDto;
import com.example.demo.api.dto.EditSysUserDto;
import com.example.demo.api.dto.QuerySysUserDto;
import com.example.demo.api.pojo.PagePojo;
import com.example.demo.api.service.ISysUserService;
import com.example.demo.framework.entity.R;
import com.example.demo.project.mpsample.domain.entity.SysUser;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.util.Assert;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * restful接口示例
 * MpTestController
 * <p>
 * @RestController("mpTestController.v1")
 *
 * @author chenzz
 * @date 2023-05-25
 */
@RestController
@RequestMapping("/user")
public class MpTestController {

    private static final Logger log = LoggerFactory.getLogger(MpTestController.class);

    @Value("${spring.application.name}")
    private String appName;

    //
    // 构造函数中注入bean
    // --------------------------------------------------------------
    private final ISysUserService userService;

    @Autowired
    public MpTestController(ISysUserService sysUserService) {
        this.userService = sysUserService;
    }
    //-----------------------

    /**
     * 列表
     * <p>
     * list(QuerySysUserDto queryDto, @RequestParam(name = "type", defaultValue = "0") Integer type)
     */
    @GetMapping("/list")
    public R list(QuerySysUserDto queryDto) {
        return R.ok().data(userService.list(queryDto));
    }

    /**
     * 分页
     */
    @GetMapping
    public R page(PagePojo page, QuerySysUserDto queryDto) {
        // 也可以使用org.springframework.data.domain.PageRequest分页对象
        Page<SysUser> listPage = userService.page(page.getPageNum(), page.getPageSize(), queryDto);
        return R.ok().data(listPage);
    }

    /**
     * 详情
     */
    @GetMapping("/{id}")
    public R detail(@PathVariable Long id) {
        SysUser detail = userService.detail(id);
        return R.ok().data(detail);
    }

    /**
     * 添加
     */
    // @Log(title = "用户", businessType = BusinessType.INSERT)
    @PostMapping
    public R create(@RequestBody AddSysUserDto dto) {
        // ValidatorUtils.validateEntity(dto);
        // Assert.isTrue(dto.getRoles().size() > 0, "角色不能为空!");
        Assert.hasText(dto.getPhonenumber(), "手机号码不能为空!");
        return userService.addSave(dto) ? R.ok() : R.error();
    }

    /**
     * 修改
     */
    @PutMapping
    public R update(@RequestBody EditSysUserDto dto) {
        Assert.notNull(dto.getUserId(), "userId不能为空!");
        return userService.updateSave(dto) ? R.ok() : R.error();
    }

    /**
     * 删除多个
     */
    @DeleteMapping
    public R remove(@RequestBody List<Long> delIdList) {
        if (delIdList.isEmpty()) {
            return R.error("There is no data to delete!");
        }
        return userService.batchRemove(delIdList) > 0 ? R.ok() : R.error();
    }

    /**
     * 删除1-n个
     * 多个用逗号隔开
     */
    @DeleteMapping("/{ids}")
    public R batchRemove(@PathVariable Long[] ids) {
        return userService.batchRemove(CollUtil.newArrayList(ids)) > 0 ? R.ok() : R.error();
    }

}
```

## BaseEntity
```java
package com.example.demo.framework.entity;

import com.baomidou.mybatisplus.annotation.TableField;
import com.fasterxml.jackson.annotation.JsonFormat;

import jakarta.persistence.Column;
import jakarta.persistence.MappedSuperclass;
import jakarta.persistence.Transient;
import java.io.Serializable;
import java.util.Date;
import java.util.Map;

import java.io.Serial;

/**
 * Entity基类
 *
 * @author chenzz
 */
@MappedSuperclass
public class BaseEntity implements Serializable {

    @Serial
    private static final long serialVersionUID = 1L;

    /**
     * 创建者
     */
    private String createBy;

    /**
     * 创建时间
     */
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date createTime;

    /**
     * 更新者
     */
    private String updateBy;

    /**
     * 更新时间
     */
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date updateTime;

    /**
     * 备注
     */
    @Column(name = "remark", columnDefinition = "longtext comment '备注'")
    private String remark;

    /**
     * 扩展数据
     */
    @TableField(exist = false)
    @Transient
    private Map<String, Object> extra;

    public String getCreateBy() {
        return createBy;
    }

    public void setCreateBy(String createBy) {
        this.createBy = createBy;
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

    public String getUpdateBy() {
        return updateBy;
    }

    public void setUpdateBy(String updateBy) {
        this.updateBy = updateBy;
    }

    public Date getUpdateTime() {
        return updateTime;
    }

    public void setUpdateTime(Date updateTime) {
        this.updateTime = updateTime;
    }

    public String getRemark() {
        return remark;
    }

    public void setRemark(String remark) {
        this.remark = remark;
    }

    public Map<String, Object> getExtra() {
        return extra;
    }

    public void setExtra(Map<String, Object> extra) {
        this.extra = extra;
    }
}
```

## entity(SysUser)
```java
package com.example.demo.project.mpsample.domain.entity;

import cn.hutool.core.date.DateUtil;
import cn.hutool.core.lang.RegexPool;
import cn.hutool.crypto.digest.DigestUtil;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableLogic;
import com.baomidou.mybatisplus.annotation.TableName;
import com.example.demo.framework.entity.BaseEntity;
import com.example.demo.framework.utils.group.UpdateGroup;
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.experimental.Accessors;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Index;
import jakarta.persistence.Table;
import jakarta.validation.constraints.*;
import java.io.Serializable;
import java.util.Date;

import java.io.Serial;

/**
 * <p>
 * 用户表 sys_user
 *
 * @author chenzz
 * @date 2023-01-08
 */

@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
@Entity()
@Table(name = "sys_user", indexes = {
        @Index(name = "index_loginName", columnList = "loginName", unique = true),
        @Index(name = "index_email", columnList = "email", unique = true),
        @Index(name = "index_phonenumber", columnList = "phonenumber", unique = true)
})
@TableName("sys_user")
public class SysUser extends BaseEntity implements Serializable {

    @Serial
    private static final long serialVersionUID = 1L;

    /**
     * 用户ID, 唯一不重复
     * 声明主键的生成策略
     */
    @Id
    @TableId(type = IdType.ASSIGN_ID)
    @JsonSerialize(using = ToStringSerializer.class)
    @NotNull(message = "修改时userId不能为空", groups = {UpdateGroup.class})
    @Min(value = 1, groups = {UpdateGroup.class})
    @Column(name = "user_id", columnDefinition = "bigint NOT NULL AUTO_INCREMENT COMMENT 'id主键'")
    private Long userId;

    /**
     * 登录账号
     */
    @Size(min = 8, max = 20, message = "账号位数必须在8-20位")
    private String loginName;

    /**
     * 用户类型（00系统用户）
     * 字符串不能为空
     * <p>
     * jakarta.validation.constraints.NotEmpty
     */
    private String userType;

    /**
     * 用户邮箱
     */
    @Pattern(message = "邮箱格式不正确!", regexp = "^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\\.[a-zA-Z0-9_-]+)+$")
    private String email;

    /**
     * 手机号码
     */
    @Pattern(message = "手机号码格式不正确!", regexp = RegexPool.MOBILE)
    private String phonenumber;

    /**
     * 用户性别（0男 1女 2未知）
     */
    private String sex;

    /**
     * 头像路径
     */
    private String avatar;

    /**
     * 密码(JsonIgnore注解字段不返回到前端)
     */
    @JsonIgnore
    private String password;

    /**
     * 盐加密
     */
    @JsonIgnore
    private String salt;

    /**
     * 帐号状态（0正常 1停用）
     */
    @Min(0)
    @Max(1)
    private Integer status;

    /**
     * 配置数据(json)
     */
    @Column(name = "config_json", columnDefinition = "json DEFAULT NULL COMMENT '配置json'")
    private String configJson;

    /**
     * 最后登陆IP
     */
    private String loginIp;

    /**
     * 最后登陆时间
     */
    private Date loginDate;

    // -----------------------------
    // -----------------------------

    /**
     * 格式化创建日期文本
     */
    public String getCreateTimeText() {
        return DateUtil.formatDateTime(this.getCreateTime());
    }

    /**
     * 格式化登录日期
     */
    public String getLoginDateText() {
        return DateUtil.formatDateTime(this.getLoginDate());
    }

    /**
     * 格式化密码
     */
    public String getEncodePwdText() {
        return DigestUtil.md5Hex(this.getPassword() + "-" + this.salt);
    }

}
```

## mapper
```java
package com.example.demo.project.mpsample.domain.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.baomidou.mybatisplus.core.toolkit.Wrappers;
import com.example.demo.project.mpsample.domain.entity.SysUser;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
import org.springframework.stereotype.Repository;

import java.util.Collections;
import java.util.List;

/**
 * 用户 mapper层
 *
 * @author chenzz
 * @date 2023-01-08
 */
@Repository
public interface SysUserMapper extends BaseMapper<SysUser> {

    /**
     * json字段(config)查询
     */
    @Select("select * from sys_user where json_extract(config,'$.type') = #{paraType}")
    List<SysUser> queryListByType(@Param("paraType") Integer type);

    /**
     * 模糊查询
     */
    @Select("select * from sys_user where username LIKE CONCAT('%',#{username},'%')")
    List<SysUser> queryListByName(String username);

    /**
     * 默认方法
     * 根据用户id(可以多个)返回用户列表
     */
    default List<SysUser> queryListByUserIds(List<Long> userIds) {
        if (userIds.isEmpty()) {
            return Collections.emptyList();
        }
        return this.selectList(Wrappers.<SysUser>lambdaQuery().in(SysUser::getUserId, userIds).orderByDesc(SysUser::getUserId));
    }

}
```

## IService
```java
package com.example.demo.api.service;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.example.demo.api.dto.AddSysUserDto;
import com.example.demo.api.dto.EditSysUserDto;
import com.example.demo.api.dto.QuerySysUserDto;
import com.example.demo.project.mpsample.domain.entity.SysUser;

import java.util.List;

/**
 * 用户 服务层
 *
 * @author chenzz
 * @date 2023-01-08
 */
public interface ISysUserService {

    /**
     * 获取查询条件Wrapper
     */
    LambdaQueryWrapper<SysUser> getLambdaQueryWrapper(QuerySysUserDto queryDto);

    /**
     * 列表
     */
    List<SysUser> list(QuerySysUserDto queryDto);

    /**
     * 分页
     */
    Page<SysUser> page(int pageNum, int pageSize, QuerySysUserDto queryDto);

    /**
     * 详情
     */
    SysUser detail(Long id);

    /**
     * 添加, 创建
     */
    boolean addSave(AddSysUserDto addDto);

    /**
     * 更新, 修改
     */
    boolean updateSave(EditSysUserDto editDto);

    /**
     * 批量删除
     *
     * @return 删除记录数量
     */
    int batchRemove(List<Long> ids);

}
```

## ServiceImpl
```java
package com.example.demo.api.service.impl;

import cn.hutool.core.collection.CollUtil;
import cn.hutool.core.date.DateTime;
import cn.hutool.core.lang.Dict;
import cn.hutool.core.util.ObjectUtil;
import cn.hutool.core.util.StrUtil;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.example.demo.api.dto.AddSysUserDto;
import com.example.demo.api.dto.EditSysUserDto;
import com.example.demo.api.dto.QuerySysUserDto;
import com.example.demo.api.service.ISysUserService;
import com.example.demo.project.mpsample.domain.entity.SysUser;
import com.example.demo.project.mpsample.domain.mapper.SysUserMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.*;
import java.util.function.BiConsumer;

/**
 * 用户信息Service业务层处理
 *
 * @author chenzz
 * @date 2023-01-08
 */
@Service
public class SysUserServiceImpl implements ISysUserService {

    private final Logger log = LoggerFactory.getLogger(getClass());

    @Autowired
    private SysUserMapper userMapper;

    @Override
    public LambdaQueryWrapper<SysUser> getLambdaQueryWrapper(QuerySysUserDto queryDto) {
        LambdaQueryWrapper<SysUser> queryWrapper = new LambdaQueryWrapper<>();

        // queryWrapper.select(SysUser::getUserId, SysUser::getLoginName, SysUser::getAvatar, SysUser::getPhonenumber);
        // ---------------------
        if (ObjectUtil.isNotNull(queryDto)) {
            queryWrapper.eq(queryDto.getUserId() != null, SysUser::getUserId, queryDto.getUserId())
                    .eq(StrUtil.isNotEmpty(queryDto.getLoginName()), SysUser::getLoginName, queryDto.getLoginName())
                    .like(StrUtil.isNotEmpty(queryDto.getPhonenumber()), SysUser::getPhonenumber, queryDto.getPhonenumber());

            // if (ObjectUtil.isAllNotEmpty(queryDto.getBeginTime(), queryDto.getEndTime())) {
            //     queryWrapper.between(SysUser::getCreateTime, queryDto.getBeginTime(), queryDto.getEndTime());
            // }
        }
        // ---------------------
        queryWrapper.orderByDesc(SysUser::getCreateTime);
        // 多字段排序, 优先级高的放前面,优先级低在后面
        // queryWrapper.orderByAsc(SysUser::getUserType, SysUser::getStatus)
        //         .orderByDesc(SysUser::getCreateTime)
        //         .groupBy(SysUser::getUserType);
        // =======================================

        log.debug("wrapper sql: {}", queryWrapper.getSqlSegment());
        log.debug("target sql: {}", queryWrapper.getTargetSql());
        // 使用Lambda表达式, 实现多线程
        new Thread(() -> {
            log.debug("thread:{}", Thread.currentThread().getName());
            // ---other---
        }).start();
        // ---------------------

        return queryWrapper;
    }

    @Override
    public List<SysUser> list(QuerySysUserDto queryDto) {
        List<SysUser> list = userMapper.selectList(getLambdaQueryWrapper(queryDto));
        // 获取手机号码集合
        // List<String> phones = list.stream().map(SysUser::getPhonenumber).filter(oo -> oo.length() == 11).collect(Collectors.toList());
        handleList(list);
        return list;
    }

    @Override
    public Page<SysUser> page(int pageNum, int pageSize, QuerySysUserDto queryDto) {
        // 分页查询, 页码从1开始
        IPage<SysUser> pageList = userMapper.selectPage(new Page<>(pageNum, pageSize), getLambdaQueryWrapper(queryDto));
        if (pageList.getTotal() > 0) {
            List<SysUser> dataList = pageList.getRecords();
            handleList(dataList);
            return new Page<SysUser>(pageList.getCurrent(), pageList.getSize(), pageList.getTotal()).setRecords(dataList);
        }
        return new Page<>(pageNum, pageSize, 0);
        // return new Page<>(pageList.getCurrent(), pageList.getSize(), pageList.getTotal());
    }

    @Override
    public SysUser detail(Long id) {
        SysUser entity = userMapper.selectById(id);
        // 设置其他扩展字段
        Dict dict = Dict.create()
                .set("e_openid", "----")
                .set("e_weixin", "~~~~")
                .set("time", DateTime.now());
        entity.setExtra(dict);
        return entity;
    }

    /**
     * 填充list字段(包括扩展数据)
     */
    private void handleList(List<SysUser> list) {
        // if (list.isEmpty()) return;
        // List<Long> userIds = list.stream().map(t -> t.getUserId()).collect(Collectors.toList());
        // List<Region> regionList = regionMapper.queryListByIdList(userIds);

        // ============(1)==============
        // 模拟生成用户扩展数据(微信登录openid等)
        List<Dict> tempDictList = genAdditionalData();
        for (SysUser mm : list) {

            // --------------------------------
            Optional<Dict> optionalAny = tempDictList.stream().filter(oo -> oo.getLong("uid").equals(mm.getUserId())).findAny();
            // 这里也可以使用ifPresent语法, 参考Stream流操作
            if (optionalAny.isPresent()) {
                Dict tempDict = optionalAny.get();

                // --------------------------------
                // 处理map数据函数, 此函数建议放到(1)位置, 不用重复执行, 提高效率, 此处仅为了演示
                BiConsumer<Map<String, Object>, Dict> consumer = (bMap, bData) -> {
                    bMap.put("e_openid", bData.getStr("openid"));
                    bMap.put("e_weixin", bData.getStr("weixin"));
                };

                // 设置扩展字段
                Map<String, Object> extra = mm.getExtra();
                if (ObjectUtil.isNotNull(extra)) {
                    // 已存在数据, 在原有数据上修改
                    consumer.accept(extra, tempDict);
                } else {
                    // 不存在
                    Map<String, Object> tempMap = new HashMap<>(16);
                    consumer.accept(tempMap, tempDict);
                    mm.setExtra(tempMap);
                }
                // ----------------
                // ----------------
                // ----------------
                // ----------------
                // Map<String, Object> extra = mm.getExtra();
                // if (ObjectUtil.isNotNull(extra)) {
                //     // 已存在数据, 在原有数据上修改
                //     extra.put("e_openid", tempDict.getStr("openid"));
                //     extra.put("e_weixin", tempDict.getStr("weixin"));
                // } else {
                //     // 不存在
                //     Map<String, Object> tempMap = new HashMap<>(16);
                //     tempMap.put("e_openid", tempDict.getStr("openid"));
                //     tempMap.put("e_weixin", tempDict.getStr("weixin"));
                //
                //     mm.setExtra(tempMap);
                // }
                // ----------------
                // ----------------
                // ----------------
            }
            // --------------------------------
        }
    }

    @Override
    public boolean addSave(AddSysUserDto addDto) {
        SysUser addEntity = new SysUser();
        BeanUtils.copyProperties(addDto, addEntity);

        Date now = new Date();
        addEntity.setCreateTime(now);
        addEntity.setUpdateTime(now);
        return userMapper.insert(addEntity) > 0;
    }

    @Override
    public boolean updateSave(EditSysUserDto editDto) {
        SysUser updateEntity = new SysUser();
        BeanUtils.copyProperties(editDto, updateEntity);

        Date now = new Date();
        updateEntity.setUpdateTime(now);
        return userMapper.updateById(updateEntity) > 0;
        // ----------------or----------------
        // LambdaUpdateWrapper<SysUser> updateWrapper = new LambdaUpdateWrapper<>();
        // if (StrUtil.isNotEmpty(editDto.getAvatar())) {
        //     updateWrapper.set(SysUser::getAvatar, editDto.getAvatar());
        // }
        // updateWrapper.set(SysUser::getUpdateTime, now)
        //         .set(SysUser::getUpdateBy, SecurityUtils.getUsernameText());
        // ---------------where--------------
        // updateWrapper.eq(SysUser::getUserId, editDto.getUserId());
        // return userMapper.update(updateWrapper) > 0;
    }

    @Override
    public int batchRemove(List<Long> ids) {
        if (ids.isEmpty()) {
            return 0;
        }
        return userMapper.deleteByIds(ids);
    }

}
```

-------------------

[spring-boot脚手架demo](https://gitee.com/chenzz/spring-boot-demo/tree/dev-framework-v3x)
