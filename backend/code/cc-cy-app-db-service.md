实际项目中使用到的app.service层数据常规操作代码, 比较适用于接口返回Dict类型数据情况, 然后结合ApiFox进行接口的调试及发布, 项目使用了java jdk17, 新语法: 默认接口, var变量推导, Function函数式接口

> mybatis-plus-boot-starter版本>=3.5.7

> 代码中使用到的ExamQuestion实体类, mapper请参看实际项目

## controller
```java
package com.cc.cy.api.controller;

import com.cc.common.annotation.Log;
import com.cc.common.core.controller.BaseController;
import com.cc.common.core.domain.AjaxResult;
import com.cc.common.core.page.PageDomain;
import com.cc.common.enums.BusinessType;
import com.cc.cy.api.dto.AddQuestion;
import com.cc.cy.api.dto.EditQuestion;
import com.cc.cy.app.service.IExamQuestionService;
import com.cc.cy.domain.entity.ExamQuestion;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.util.Assert;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * 在线考试.考题
 *
 * @author chenzz
 * @date 2023-11-25
 */
@RestController("examQuestionController.v1")
@RequestMapping("/cc/exam")
public class ExamQuestionController extends BaseController {

    @Autowired
    private IExamQuestionService examQuestionService;

    /**
     * 考题分页
     */
    @GetMapping("/question-page")
    public AjaxResult questionPage(PageDomain page, ExamQuestion queryDto) {
        return success(examQuestionService.page(page.getPageNum(), page.getPageSize(), queryDto));
    }

    /**
     * 考题详情
     */
    @GetMapping("/question-detail/{id}")
    public AjaxResult questionDetail(@PathVariable Long id) {
        return success(examQuestionService.detailDict(id));
    }

    /**
     * 添加单选考题
     */
    @Log(title = "添加单选考题", businessType = BusinessType.INSERT)
    @PostMapping("/add-single-choice-question")
    public AjaxResult createSingleChoiceQuestion(@RequestBody AddQuestion<AddQuestion.ChoiceItem> dto) {
        Assert.hasText(dto.getTitle(), "题干不能为空!");
        Assert.notEmpty(dto.getContent(), "题目选项不能为空!");
        Assert.notNull(dto.getAnswer(), "答案不能为空!");

        return toAjax(examQuestionService.createTtQuestion(dto, 1));
    }

    /**
     * 修改单选考题
     */
    @Log(title = "修改单选考题", businessType = BusinessType.UPDATE)
    @PutMapping("/edit-single-choice-question")
    public AjaxResult editSingleChoiceQuestion(@RequestBody EditQuestion<AddQuestion.ChoiceItem> dto) {
        Assert.notNull(dto.getId(), "id不能为空!");
        return toAjax(examQuestionService.editTtQuestion(dto.getId(), dto));
    }

    /**
     * 删除1-n个
     */
    @Log(title = "删除考题", businessType = BusinessType.DELETE)
    @DeleteMapping("/remove-question/{ids}")
    public AjaxResult batchRemove(@PathVariable Long[] ids) {
        return toAjax(examQuestionService.batchRemove(List.of(ids)));
    }

}
```


## addDto(泛型类)
```java
package com.cc.cy.api.dto;

import lombok.Getter;
import lombok.Setter;

import java.io.Serial;
import java.io.Serializable;
import java.util.List;

/**
 * 添加单选题, 试题/题库
 * 泛型类用于适配答案(单选, 多选)
 *
 * <p>
 * 在一个类的有参构造函数中调用无参构造函数, 是通过使用this()关键字来实现的. 需要放在第一行
 * <p>
 * eg: AddQuestion<List<String>> dto, EditQuestion<AddQuestion.ChoiceItem> dto
 *
 * @author chenzz
 */
@Setter
@Getter
public class AddQuestion<T> implements Serializable {

    @Serial
    private static final long serialVersionUID = 1L;

    public AddQuestion() {
        // 利用无参构造函数初始化变量默认值
        this.score = 5;
    }

    /**
     * 题干/题目
     */
    private String title;

    /**
     * 题目选项
     */
    private List<ChoiceItem> content;

    /**
     * 答案
     * 单选/多选/判断的答案不一样, 所以使用泛型
     */
    private T answer;

    /**
     * 讲解
     */
    private String answerExplain;

    /**
     * 分数
     */
    private Integer score;

    @Setter
    @Getter
    public static class ChoiceItem {

        /**
         * 序号
         */
        private String no;

        /**
         * 内容
         */
        private String text;
    }

}
```

## editDto(继承自addDto)
```java
package com.cc.cy.api.dto;

import lombok.Getter;
import lombok.Setter;

/**
 * 修改单选题, 试题/题库
 *
 * @author chenzz
 */
@Getter
@Setter
public class EditQuestion<T> extends AddQuestion<T> {
    private Long id;

    public EditQuestion() {
        super();
    }

}
```


## interface
```java
package com.cc.cy.app.service;

import cn.hutool.core.lang.Dict;
import cn.hutool.core.util.ObjectUtil;
import cn.hutool.json.JSONConfig;
import cn.hutool.json.JSONObject;
import cn.hutool.json.JSONUtil;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.cc.cy.api.dto.AddQuestion;
import com.cc.cy.api.dto.EditQuestion;
import com.cc.cy.domain.entity.ExamQuestion;

import java.util.Collections;
import java.util.List;
import java.util.function.Function;

/**
 * 试题/题目Service接口
 *
 * @author chenzz
 * @date 2023-11-25
 */
public interface IExamQuestionService {

    /**
     * 接口中定义转换函数方便其他地方调用(默认就是public final)
     * <p>
     * Entity -> Dict
     */
    Function<ExamQuestion, Dict> funConvert = (entity) -> {
        Dict dict = Dict.create();
        dict.set("id", String.valueOf(entity.getId()));
        dict.set("remark", entity.getRemark());

        dict.set("createBy", entity.getCreateBy());
        // dict.set("createTime", DateUtil.formatDateTime(entity.getCreateTime()));

        dict.set("updateBy", entity.getUpdateBy());
        // dict.set("updateTime", DateUtil.formatDateTime(entity.getUpdateTime()));
        // --------------------------------
        // --------------------------------

        dict.set("title", entity.getTitle());
        // --------------------------------

        return dict;
    };

    /**
     * 获取查询条件Wrapper
     */
    LambdaQueryWrapper<ExamQuestion> getLambdaQueryWrapper(ExamQuestion queryDto);

    /**
     * 列表
     */
    List<ExamQuestion> list(ExamQuestion queryDto);

    /**
     * 分页
     */
    Page<Dict> page(int pageNum, int pageSize, ExamQuestion queryDto);

    /**
     * 详情
     */
    ExamQuestion detail(Long id);

    /**
     * 详情(Dict)
     */
    Dict detailDict(Long id);

    /**
     * 添加|创建, 返回刚添加的对象(可以通过对象获取自增id)
     */
    ExamQuestion addSaveForEntity(ExamQuestion addEntity);

    /**
     * 更新|修改
     */
    boolean updateSave(ExamQuestion editDto);

    /**
     * 批量删除
     *
     * @return 删除记录数量
     */
    int batchRemove(List<Long> ids);

    /**
     * 添加|创建
     *
     * @return 是否添加成功
     */
    default boolean addSave(ExamQuestion addEntity) {
        return ObjectUtil.isNotNull(addSaveForEntity(addEntity));
    }

    /**
     * 转换成Dict集合返回
     */
    default List<Dict> listDict(ExamQuestion queryDto) {
        // ---------------------------------
        return list(queryDto).stream().map(oo -> {
            Dict ooDict = funConvert.apply(oo);
            ooDict.set("html_page", "http://xxx.top/xxx/100");
            return ooDict;
        }).toList();

        // ---------------------------------
        // ExamQuestionMapper mapper = SpringUtils.getBean(ExamQuestionMapper.class);
        // --------------
        // var list = mapper.selectList(getLambdaQueryWrapper(queryDto));
        // return list.stream().map(funConvert).toList();
    }

    /**
     * 添加考题(单选, 多选, 判断等), 泛型方法
     * <p>
     * AddQuestion<AddQuestion.ChoiceItem> dto;
     * examQuestionService.createTtQuestion(dto, 1);
     */
    default <T> boolean createTtQuestion(AddQuestion<T> dto, Integer questionType) {
        ExamQuestion addEntity = new ExamQuestion();
        addEntity.setTitle(dto.getTitle());
        addEntity.setScore(dto.getScore());
        addEntity.setAnswerExplain(dto.getAnswerExplain());
        addEntity.setQuestionType(questionType);
        addEntity.setStatus(0);

        JSONConfig jsonConfig = JSONConfig.create().setIgnoreNullValue(false).setStripTrailingZeros(false);
        String jsonChoice = JSONUtil.toJsonStr(dto.getContent(), jsonConfig);
        String jsonAnswer = JSONUtil.toJsonStr(dto.getAnswer(), jsonConfig);
        addEntity.setContent(jsonChoice);
        addEntity.setAnswer(jsonAnswer);

        // 题目配置
        JSONObject jsonObject = genJsonObjByDto(dto);
        addEntity.setQuestionConfig(jsonObject.toString());

        return addSave(addEntity);
    }

    /**
     * 修改考题(单选, 多选, 判断等)
     * <p>
     * EditQuestion<AddQuestion.ChoiceItem> dto;
     * examQuestionService.editTtQuestion(1L, dto);
     */
    default <T> boolean editTtQuestion(Long id, EditQuestion<T> dto) {
        ExamQuestion editDto = new ExamQuestion();
        editDto.setTitle(dto.getTitle());
        editDto.setScore(dto.getScore());
        editDto.setAnswerExplain(dto.getAnswerExplain());

        JSONConfig jsonConfig = JSONConfig.create().setIgnoreNullValue(false).setStripTrailingZeros(false);
        String jsonChoice = JSONUtil.toJsonStr(dto.getContent(), jsonConfig);
        String jsonAnswer = JSONUtil.toJsonStr(dto.getAnswer(), jsonConfig);
        editDto.setContent(jsonChoice);
        editDto.setAnswer(jsonAnswer);

        // 题目配置
        JSONObject jsonObject = genJsonObjByDto(dto);
        editDto.setQuestionConfig(jsonObject.toString());
        // ---------------
        editDto.setId(id);

        return updateSave(editDto);
    }

    /**
     * 接口中的私有方法(不能有公共方法)
     */
    private <T> JSONObject genJsonObjByDto(AddQuestion<T> dto) {
        JSONObject jsonObject = new JSONObject();
        jsonObject.set("knowledge", dto.getKnowledge());
        jsonObject.set("dimension", List.of(dto.getKnowledge()));
        jsonObject.set("tags", Collections.emptyList());

        jsonObject.set("subCategory", dto.getSubCategory());
        return jsonObject;
    }

}
```

## ServiceImpl
```java
package com.cc.cy.app.service.impl;

import cn.hutool.core.date.DateUtil;
import cn.hutool.core.lang.Dict;
import cn.hutool.core.util.ObjectUtil;
import cn.hutool.core.util.StrUtil;
import cn.hutool.json.JSONUtil;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.LambdaUpdateWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.cc.common.utils.SecurityUtils;
import com.cc.cy.app.service.IExamQuestionService;
import com.cc.cy.domain.entity.ExamQuestion;
import com.cc.cy.domain.mapper.ExamQuestionMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.Date;
import java.util.List;
import java.util.function.Function;
// import com.baomidou.mybatisplus.core.toolkit.Wrappers;

/**
 * 试题/题目Service实现
 *
 * @author chenzz
 * @date 2023-11-25
 */
@Service
public class ExamQuestionServiceImpl implements IExamQuestionService {

    private final Logger log = LoggerFactory.getLogger(ExamQuestionServiceImpl.class);

    @Autowired
    private ExamQuestionMapper examQuestionMapper;

    /**
     * 如果只需在当前类内部调用, 推荐在类内部定义为private, 反之放到接口中定义(这样在当前类也一样可以调用)
     * <p>
     * 这里funConvert会覆盖接口中的同名变量, 本ExamQuestionServiceImpl类中的调用将使用这里的实现
     *
     * Entity -> Dict
     */
    private final Function<ExamQuestion, Dict> funConvert = (entity) -> {
        Dict dict = Dict.create();

        int questionType = entity.getQuestionType();
        String content = entity.getContent();
        String answer = entity.getAnswer();

        dict.set("id", String.valueOf(entity.getId()));
        dict.set("questionType", questionType);
        dict.set("title", entity.getTitle());
        dict.set("answerExplain", entity.getAnswerExplain());
        dict.set("score", entity.getScore());
        dict.set("createTime", DateUtil.formatDateTime(entity.getCreateTime()));

        dict.set("content", JSONUtil.parse(content));
        dict.set("answer", JSONUtil.parse(answer));
        return dict;
    };

    @Override
    public LambdaQueryWrapper<ExamQuestion> getLambdaQueryWrapper(ExamQuestion queryDto) {
        LambdaQueryWrapper<ExamQuestion> queryWrapper = new LambdaQueryWrapper<>();
        // queryWrapper.select(ExamQuestion::getId, ExamQuestion::getQuestionType, ExamQuestion::getTitle, ExamQuestion::getContent);
        // ---------------------
        if (ObjectUtil.isNotNull(queryDto)) {
            queryWrapper.eq(queryDto.getId() != null, ExamQuestion::getId, queryDto.getId())
                    .eq(ObjectUtil.isNotNull(queryDto.getQuestionType()), ExamQuestion::getQuestionType, queryDto.getQuestionType())
                    .eq(ObjectUtil.isNotNull(queryDto.getStatus()), ExamQuestion::getStatus, queryDto.getStatus())
                    .like(StrUtil.isNotEmpty(queryDto.getTitle()), ExamQuestion::getTitle, queryDto.getTitle());

            // 例1: apply("id = 1")
            // 例2: apply("date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")
            // 例3: apply("date_format(dateColumn,'%Y-%m-%d') = {0}", LocalDate.now())
            // 例4: apply("name={0,javaType=int,jdbcType=NUMERIC,typeHandler=xxx.xxx.MyTypeHandler}", "老王")
            queryWrapper.apply("JSON_CONTAINS(config_json,JSON_OBJECT('tags', {0}))", "'国产'");        
        }
        // ---------------------
        queryWrapper.orderByDesc(ExamQuestion::getCreateTime);
        return queryWrapper;
    }

    @Override
    public List<ExamQuestion> list(ExamQuestion queryDto) {
        List<ExamQuestion> list = examQuestionMapper.selectList(getLambdaQueryWrapper(queryDto));
        handleList(list);
        return list;

        // ---------------------------------
        // var list = examQuestionMapper.selectList(getLambdaQueryWrapper(queryDto));
        // var dictList = list.stream().map(oo -> {
        //     Dict ooDict = funConvert.apply(oo);
        //     ooDict.set("html_page", "http://xxx.top/xxx/100");
        //     return ooDict;
        // }).toList();
        // return dictList;
    }

    @Override
    public Page<Dict> page(int pageNum, int pageSize, ExamQuestion queryDto) {
        // 分页查询, 页码从1开始
        IPage<ExamQuestion> pageList = examQuestionMapper.selectPage(new Page<>(pageNum, pageSize), getLambdaQueryWrapper(queryDto));
        if (pageList.getTotal() > 0) {
            List<ExamQuestion> dataList = pageList.getRecords();
            List<Dict> dictList = dataList.stream().map(funConvert).toList();
            return new Page<Dict>(pageList.getCurrent(), pageList.getSize(), pageList.getTotal()).setRecords(dictList);
        }
        return new Page<>(pageNum, pageSize, 0);
    }

    @Override
    public ExamQuestion detail(Long id) {
        return examQuestionMapper.selectById(id);
    }

    @Override
    public Dict detailDict(Long id) {
        return funConvert.apply(examQuestionMapper.selectById(id));

        // ---------------------------------
        // ExamQuestion entityDetail = examQuestionMapper.selectById(id);
        // Dict xxDict = funConvert.apply(entityDetail);

        // 设置其他数据字段
        // 可以进一步关联其他表对Dict添加其他字段数据(比较便于扩展) 
        // --------------------
        // xxDict.set("html_page", "http://xxx.top/xxx/100");
        // return xxDict;
    }

    /**
     * 根据list填充字段(包括扩展数据)
     */
    private void handleList(List<ExamQuestion> list) {
        for (var mm : list) {
            mm.setExtra(Dict.create().set("remark", "备注"));
        }
    }

    @Override
    public ExamQuestion addSaveForEntity(ExamQuestion addEntity) {
        String loginUserName = SecurityUtils.getUsernameText();
        addEntity.setCreateBy(loginUserName);
        addEntity.setUpdateBy(loginUserName);

        Date now = new Date();
        addEntity.setCreateTime(now);
        addEntity.setUpdateTime(now);

        int result = examQuestionMapper.insert(addEntity);
        return result > 0 ? addEntity : null;
    }

    @Override
    public boolean updateSave(ExamQuestion editDto) {
        Date now = new Date();

        LambdaUpdateWrapper<ExamQuestion> updateWrapper = new LambdaUpdateWrapper<>();
        if (StrUtil.isNotEmpty(editDto.getTitle())) {
            updateWrapper.set(ExamQuestion::getTitle, editDto.getTitle());
        }
        if (JSONUtil.isTypeJSONObject(editDto.getQuestionConfig())) {
            // 题目属性配置(json)
            // {"tags": ["视频"], "subject": "运营", "dimension": ["运营基础"], "knowledge": "运营基础"}
            updateWrapper.set(ExamQuestion::getQuestionConfig, editDto.getQuestionConfig());
        }
        if (JSONUtil.isTypeJSONArray(editDto.getContent())) {
            updateWrapper.set(ExamQuestion::getContent, editDto.getContent());
        }
        if (JSONUtil.isTypeJSON(editDto.getAnswer())) {
            updateWrapper.set(ExamQuestion::getAnswer, editDto.getAnswer());
            // 更新JSON字段
            // UPDATE tb_question SET config_json = JSON_SET(config_json, '$.tags', JSON_ARRAY('java', 'javascript', 'c')) WHERE id=1;
            updateWrapper.setSql(cn.hutool.core.util.StrUtil.format("config_json = JSON_SET(config_json, '$.type', {})", 99));
        }
        updateWrapper.set(ExamQuestion::getUpdateTime, now)
                .set(ExamQuestion::getUpdateBy, SecurityUtils.getUsernameText());
        // ---------------where--------------
        updateWrapper.eq(ExamQuestion::getId, editDto.getId());
        return examQuestionMapper.update(updateWrapper) > 0;
    }

    @Override
    public int batchRemove(List<Long> ids) {
        if (ids.isEmpty()) {
            return 0;
        }
        // --------------------------------------------
        // 非管理员只能删除本人数据!
        // long userId = SecurityUtils.getUserId();
        // List<ExamQuestion> list = examQuestionMapper.queryListByIdList(ids);
        // list.forEach(oo -> {
        //     if (userId > 1 && oo.getUserId() != userId) {
        //         throw new ServiceException("只允许删除本人数据!");
        //     }
        // });
        // --------------------------------------------
        // --------------------------------------------
        // int deleteRows = examQuestionMapper.delete(Wrappers.<ExamQuestion>lambdaQuery().eq(ExamQuestion::getStatus, 0).in(ExamQuestion::getId, ids));
        return examQuestionMapper.deleteByIds(ids);
    }

}
```

## 从分页/列表请求中获取时间段
```java
import cn.hutool.core.date.DateField;
import cn.hutool.core.date.DateTime;
import cn.hutool.core.date.DateUtil;
import cn.hutool.core.util.StrUtil;
import com.cc.common.utils.ServletUtils;

// 从分页/列表请求参数中获取时间段参数, 一般为GET请求
// ===================================
String beginTime = ServletUtils.getParameter("params[beginTime]");
String endTime = ServletUtils.getParameter("params[endTime]");

// 计算起始时间
DateTime mmBeginTime;
DateTime mmEndTime;

// 程序用到的时间(适当放宽)
DateTime javaBeginTime;
DateTime javaEndTime;

if (StrUtil.isAllNotEmpty(beginTime, endTime)) {
    // 起始时间都有传
    mmBeginTime = DateUtil.beginOfDay(DateUtil.parseDate(beginTime));
    mmEndTime = DateUtil.endOfDay(DateUtil.parseDate(endTime));

    // 在客户端时间基础上放宽8,2天
    javaBeginTime = mmBeginTime.offsetNew(DateField.DAY_OF_MONTH, -8);
    javaEndTime = mmEndTime.offsetNew(DateField.DAY_OF_MONTH, 2);
} else {
    // 未传参数, 设置默认值
    mmBeginTime = DateUtil.beginOfWeek(DateTime.now());
    mmEndTime = DateTime.now();

    javaBeginTime = mmBeginTime.offsetNew(DateField.DAY_OF_MONTH, -8);
    javaEndTime = mmEndTime;
}

// beginTime:2023-12-01, endTime:2024-01-04, mmBeginTime:2023-12-01 00:00:00, mmEndTime:2024-01-04 23:59:59, javaBeginTime:2023-11-23 00:00:00, javaEndTime:2024-01-06 23:59:59
log.debug("beginTime:{}, endTime:{}, mmBeginTime:{}, mmEndTime:{}, javaBeginTime:{}, javaEndTime:{}", beginTime, endTime, DateUtil.formatDateTime(mmBeginTime), DateUtil.formatDateTime(mmEndTime), DateUtil.formatDateTime(javaBeginTime), DateUtilformatDateTime(javaEndTime));
// ===================================
```