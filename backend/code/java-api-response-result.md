## 返回HashMap, 方便动态添加, 修改字段
```java
package com.example.demo.entity;

import java.io.Serial;
import java.util.HashMap;

/**
 * 接口返回对象
 * eg: R.ok().data(list)
 *
 * @author chenzz
 */
public class R extends HashMap<String, Object> {

    @Serial
    private static final long serialVersionUID = 1L;

    private static final int SUCCESS = 0;

    private static final int FAIL = 500;

    public R() {
        put("code", SUCCESS);
        put("msg", "success");
    }

    public static R error() {
        return error(FAIL, "操作失败,未知异常");
    }

    public static R error(String msg) {
        return error(FAIL, msg);
    }

    public static R error(int code, String msg) {
        R r = new R();
        r.put("code", code);
        r.put("msg", msg);
        return r;
    }

    public static R ok(String msg) {
        R r = new R();
        r.put("msg", msg);
        return r;
    }

    public static R ok() {
        return new R();
    }

    //-------------------------------

    public R data(Object o) {
        return put("data", o);
    }

    @Override
    public R put(String key, Object value) {
        super.put(key, value);
        return this;
    }

}
```


## 范型返回结果类, 通过泛型指定返回数据类型
```java
package com.example.demo.entity;

import java.io.Serial;
import java.io.Serializable;

/**
 * 响应信息主体
 * eg: Results.ok(Collections.emptyList())
 *
 * @author chenzz
 */
public class Results<T> implements Serializable {

    @Serial
    private static final long serialVersionUID = 1L;

    /**
     * 成功
     */
    private static final int SUCCESS = 0;

    /**
     * 失败
     */
    private static final int FAIL = 500;

    /**
     * 成功
     */
    private static final String SUCCESS_MSG = "success";

    /**
     * 失败
     */
    private static final String FAIL_MSG = "fail";

    private int code;

    private String msg;

    private T data;

    public static <T> Results<T> ok() {
        return restResult(null, SUCCESS, SUCCESS_MSG);
    }

    public static <T> Results<T> ok(T data) {
        return restResult(data, SUCCESS, SUCCESS_MSG);
    }

    public static <T> Results<T> ok(T data, String msg) {
        return restResult(data, SUCCESS, msg);
    }

    //=============================================

    public static <T> Results<T> fail() {
        return restResult(null, FAIL, FAIL_MSG);
    }

    public static <T> Results<T> fail(String msg) {
        return restResult(null, FAIL, msg);
    }

    public static <T> Results<T> fail(T data) {
        return restResult(data, FAIL, FAIL_MSG);
    }

    public static <T> Results<T> fail(T data, String msg) {
        return restResult(data, FAIL, msg);
    }

    public static <T> Results<T> fail(int code, String msg) {
        return restResult(null, code, msg);
    }

    //=============================================

    private static <T> Results<T> restResult(T data, int code, String msg) {
        Results<T> apiResult = new Results<>();
        apiResult.setCode(code);
        apiResult.setData(data);
        apiResult.setMsg(msg);
        return apiResult;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public Boolean isError() {
        return !isSuccess();
    }

    public Boolean isSuccess() {
        return Results.SUCCESS == getCode();
    }
}
```