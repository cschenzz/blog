controller, interface, impl实现示例代码

> mybatis-plus-boot-starter版本>=3.5.7

## controller
```java
package com.cc.oox.api.controller;

import com.cc.common.annotation.Log;
import com.cc.common.core.controller.BaseController;
import com.cc.common.core.domain.AjaxResult;
import com.cc.common.core.page.PageDomain;
import com.cc.common.enums.BusinessType;
import com.cc.oox.api.dto.AddServiceRegionDto;
import com.cc.oox.api.dto.EditServiceRegionDto;
import com.cc.oox.app.service.IServiceRegionService;
import com.cc.oox.domain.entity.ServiceRegion;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.util.Assert;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * api-服务区域
 *
 * @author chenzz
 */
@RestController
@RequestMapping("/yy/region")
public class ServiceRegionController extends BaseController {

    @Autowired
    private IServiceRegionService serviceRegionService;

    /**
     * 分页
     */
    @GetMapping
    public AjaxResult page(PageDomain page, ServiceRegion queryDto) {
        return success(serviceRegionService.page(page.getPageNum(), page.getPageSize(), queryDto));
    }

    /**
     * 详情
     */
    @GetMapping("/{id}")
    public AjaxResult detail(@PathVariable Long id) {
        return success(serviceRegionService.detailDict(id));
    }

    /**
     * 添加
     */
    @Log(title = "服务区域", businessType = BusinessType.INSERT)
    @PostMapping
    public AjaxResult create(@RequestBody AddServiceRegionDto dto) {
        // validateEntity(dto);
        Assert.hasText(dto.getName(), "名称不能为空!");
        return toAjax(serviceRegionService.createRegion(dto));
    }

    /**
     * 修改
     */
    @Log(title = "服务区域", businessType = BusinessType.UPDATE)
    @PutMapping
    public AjaxResult edit(@RequestBody EditServiceRegionDto dto) {
        Assert.notNull(dto.getId(), "id不能为空!");
        validateEntity(dto);
        return toAjax(serviceRegionService.editRegion(dto.getId(), dto));
    }

    /**
     * 删除1-n个
     */
    @Log(title = "服务区域", businessType = BusinessType.DELETE)
    @PreAuthorize("@ss.isAdmin()")
    @DeleteMapping("/{ids}")
    public AjaxResult batchRemove(@PathVariable Long[] ids) {
        return toAjax(serviceRegionService.batchRemove(List.of(ids)));
    }

}
```


## interface
```java
package com.cc.oox.app.service;

import cn.hutool.core.date.DateUtil;
import cn.hutool.core.lang.Dict;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.cc.common.utils.spring.SpringUtils;
import com.cc.oox.api.dto.AddServiceRegionDto;
import com.cc.oox.api.dto.EditServiceRegionDto;
import com.cc.oox.domain.entity.ServiceRegion;
import com.cc.oox.domain.mapper.ServiceRegionMapper;
import org.springframework.beans.BeanUtils;

import java.util.List;
import java.util.function.Function;

/**
 * 服务区域Service接口
 *
 * @author chenzz
 */
public interface IServiceRegionService {

    /**
     * Entity -> Dict
     */
    Function<ServiceRegion, Dict> funConvert = (entity) -> {
        Dict dict = Dict.create();
        dict.set("id", String.valueOf(entity.getId()));
        dict.set("createBy", entity.getCreateBy());
        dict.set("createTime", DateUtil.formatDateTime(entity.getCreateTime()));

        dict.set("remark", entity.getRemark());
        dict.set("updateBy", entity.getUpdateBy());
        dict.set("updateTime", DateUtil.formatDateTime(entity.getUpdateTime()));
        // --------------------------------

        dict.set("name", entity.getName());
        // --------------------------------
        return dict;
    };

    /**
     * 获取查询条件Wrapper
     */
    LambdaQueryWrapper<ServiceRegion> getLambdaQueryWrapper(ServiceRegion queryDto);

    /**
     * 分页
     */
    Page<Dict> page(int pageNum, int pageSize, ServiceRegion queryDto);

    /**
     * 详情
     */
    ServiceRegion detail(Long id);

    /**
     * 详情(Dict)
     */
    Dict detailDict(Long id);

    /**
     * 添加|创建
     */
    boolean addSave(ServiceRegion addEntity);

    /**
     * 更新|修改
     */
    boolean updateSave(ServiceRegion editDto);

    /**
     * 批量删除
     *
     * @return 删除记录数量
     */
    int batchRemove(List<Long> ids);

    /**
     * 获取Mapper
     */
    default ServiceRegionMapper getMapper() {
        return SpringUtils.getBean(ServiceRegionMapper.class);
    }

    /**
     * 列表
     */
    default List<Dict> list(ServiceRegion queryDto) {
        ServiceRegionMapper mapper = getMapper();
        // --------------
        return mapper.selectList(getLambdaQueryWrapper(queryDto)).stream().map(funConvert).toList();
    }

    /**
     * 添加一条新数据
     */
    default boolean createRegion(AddServiceRegionDto dto) {
        ServiceRegion addEntity = new ServiceRegion();
        BeanUtils.copyProperties(dto, addEntity);

        // -----------------
        addEntity.setName(dto.getName());
        // -----------------

        return addSave(addEntity);
    }

    /**
     * 根据id修改
     */
    default boolean editRegion(Long id, EditServiceRegionDto dto) {
        ServiceRegion editDto = new ServiceRegion();
        BeanUtils.copyProperties(dto, editDto);
        // ---------------
        editDto.setName(dto.getName());
        // ---------------
        editDto.setId(id);

        return updateSave(editDto);
    }

}
```

## ServiceImpl
```java
package com.cc.oox.app.service.impl;

import cn.hutool.core.lang.Dict;
import cn.hutool.core.util.ObjectUtil;
import cn.hutool.core.util.StrUtil;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.LambdaUpdateWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.cc.common.utils.SecurityUtils;
import com.cc.oox.app.service.IServiceRegionService;
import com.cc.oox.domain.entity.ServiceRegion;
import com.cc.oox.domain.mapper.ServiceRegionMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.Date;
import java.util.List;


/**
 * 服务区域Service实现
 *
 * @author chenzz
 */
@Service
public class ServiceRegionServiceImpl implements IServiceRegionService {

    private final Logger log = LoggerFactory.getLogger(ServiceRegionServiceImpl.class);

    @Autowired
    private ServiceRegionMapper serviceRegionMapper;

    @Override
    public LambdaQueryWrapper<ServiceRegion> getLambdaQueryWrapper(ServiceRegion queryDto) {
        LambdaQueryWrapper<ServiceRegion> queryWrapper = new LambdaQueryWrapper<>();
        // ---------------------
        if (ObjectUtil.isNotNull(queryDto)) {
            queryWrapper.eq(ObjectUtil.isNotNull(queryDto.getId()), ServiceRegion::getId, queryDto.getId())
                    .like(StrUtil.isNotEmpty(queryDto.getName()), ServiceRegion::getName, queryDto.getName());
        }
        // ---------------------
        queryWrapper.orderByAsc(ServiceRegion::getCreateTime);
        return queryWrapper;
    }

    @Override
    public Page<Dict> page(int pageNum, int pageSize, ServiceRegion queryDto) {
        IPage<ServiceRegion> pageList = serviceRegionMapper.selectPage(new Page<>(pageNum, pageSize), getLambdaQueryWrapper(queryDto));
        if (pageList.getTotal() > 0) {
            List<ServiceRegion> dataList = pageList.getRecords();
            List<Dict> dictList = dataList.stream().map(funConvert).toList();
            return new Page<Dict>(pageList.getCurrent(), pageList.getSize(), pageList.getTotal()).setRecords(dictList);
        }
        return new Page<>(pageNum, pageSize, 0);
    }

    @Override
    public ServiceRegion detail(Long id) {
        return serviceRegionMapper.selectById(id);
    }

    @Override
    public Dict detailDict(Long id) {
        return funConvert.apply(serviceRegionMapper.selectById(id));
    }

    @Override
    public boolean addSave(ServiceRegion addEntity) {
        String loginUserName = SecurityUtils.getUsernameText();
        addEntity.setCreateBy(loginUserName);
        addEntity.setUpdateBy(loginUserName);

        Date now = new Date();
        addEntity.setCreateTime(now);
        addEntity.setUpdateTime(now);
        return serviceRegionMapper.insert(addEntity) > 0;

        // -------------------------
        // 需要获取自增id时使用下面方法
        // int result = serviceRegionMapper.insert(addEntity);
        // return result > 0 ? addEntity.getId() : -100L;
    }

    @Override
    public boolean updateSave(ServiceRegion editDto) {
        Date now = new Date();

        LambdaUpdateWrapper<ServiceRegion> updateWrapper = new LambdaUpdateWrapper<>();
        if (StrUtil.isNotEmpty(editDto.getName())) {
            updateWrapper.set(ServiceRegion::getName, editDto.getName());
        }
        if (ObjectUtil.isNotNull(editDto.getZoneId())) {
            updateWrapper.set(ServiceRegion::getZoneId, editDto.getZoneId());
        }

        updateWrapper.set(ServiceRegion::getUpdateTime, now)
                .set(ServiceRegion::getUpdateBy, SecurityUtils.getUsernameText());
        // ---------------where--------------
        updateWrapper.eq(ServiceRegion::getId, editDto.getId());
        return serviceRegionMapper.update(updateWrapper) > 0;

        // --------更新JSON字段(array)--------
        // var tags = List.of("java", "javascript", "python", "c");
        // var __tags = tags.stream().map(oo -> "'" + oo + "'").toList();
        // serviceRegionMapper.update(Wrappers.<ServiceRegion>lambdaUpdate()
        //        .setSql(cn.hutool.core.util.StrUtil.format("config_json = JSON_SET(config_json, '$.tags', JSON_ARRAY({}))", String.join(",", __tags)))
        //        .set(ServiceRegion::getUpdateTime, LocalDateTime.now())
        //        .eq(ServiceRegion::getId, 1));

        // ----------------------------------
        // ----------------------------------
    }

    @Override
    public int batchRemove(List<Long> ids) {
        if (ids.isEmpty()) {
            return 0;
        }
        return serviceRegionMapper.deleteByIds(ids);
    }

}
```