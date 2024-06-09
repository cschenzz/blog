## token
```java
import cn.hutool.core.date.DateUtil;
import cn.hutool.core.date.TimeInterval;
import cn.hutool.core.lang.Console;
import cn.hutool.core.lang.Dict;
import cn.hutool.http.HttpRequest;
import cn.hutool.http.HttpResponse;
import cn.hutool.json.JSONUtil;

import java.io.BufferedOutputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.File;


private final String token = "token";
```

## testGet
```java
String url = "http://127.0.0.1:8888" + "/api/app/v1/MyReceiveAddress";
// ================================
String result = HttpRequest.get(url)
        .header("Authorization", token)
        .execute().body();
String prettyJson = JSONUtil.toJsonPrettyStr(result);
Console.log("{}{}-------------------------", url, System.lineSeparator());
Console.log(prettyJson);
```

## get请求时form传参测试
```java
String url = "http://127.0.0.1:8888" + "/api/app/v1/MyReceiveAddress";
// ================================
// get请求时params参数可作为url传参形式(?p=2&s=15&keyWords=china)
// post请求时params参数可作为form传参数形式
Dict params = Dict.create()
        .set("p", 2)
        .set("s", 15)
        .set("keyWords", "china");
HttpResponse response = HttpRequest.get(url)
        .header("Authorization", token)
        .form(params)
        .execute();
int status = response.getStatus();
String result = response.body();
String prettyJson = JSONUtil.toJsonPrettyStr(result);
Console.log("(status: {}): {}", status, url);
Console.log("-------------------------------------");
Console.log(prettyJson);
```

## post-向服务器发送json数据
```java
// insert, add
String url = "http://127.0.0.1:8888" + "/api/app/v1/addReceiveAddress";
// ---------------------
Dict dt = Dict.create()
        .set("addressDetail", "软件园1期B01栋")
        .set("defaultAddress", true)
        .set("districtId", 350211)
        .set("receiveMobileNo", "13800138000")
        .set("receiveUserName", "陈某某");
String json = JSONUtil.toJsonStr(dt);
// ================================
String result = HttpRequest.post(url)
        .header("Authorization", token)
        .body(json)
        .execute().body();
Console.log(result);
```

## post(form-data)提交数据
```java
String url = "https://api.xxx.com/test-xxx";
Dict dt = Dict.create()
        .set("name", "李白")
        .set("age", 23);

String result = HttpRequest.post(url)
        .header("Authorization", "token-xxx")
        .form(dt)
        .execute().body();
Console.log(result);
```

## Put
```java
// update, edit
String url = "http://127.0.0.1:8888" + "/api/app/v1/updateReceiveAddress";
// ---------------------
Dict dt = Dict.create()
        .set("id", "11")
        .set("addressDetail", "软件园1期B01")
        // .set("defaultAddress", true)
        // .set("districtId", 110101)
        // .set("districtText", "北京市市辖区东城区")
        .set("receiveMobileNo", "13800138000")
        .set("receiveUserName", "猪八戒");
String json = JSONUtil.toJsonStr(dt);
// ================================
String result = HttpRequest.put(url)
        .header("Authorization", token)
        .body(json)
        .execute().body();
Console.log(result);
```

## Delete
```java
// delete
String url = "http://127.0.0.1:8888" + "/api/app/v1/deleteReceiveAddress/115";
// ================================
String result = HttpRequest.delete(url)
        .header("Authorization", token)
        .header("User-Agent", "Hutool.Http")
        .execute().body();
Console.log(result);
```

## post-上传文件
```java
// form上传文件支持多个
String url = "http://127.0.0.1:8888/file";
String result = HttpRequest.post(url)
        .form("file", FileUtil.file("/home/chenzz/file/ff-01.zip"))
        .execute().body();
Console.log(result);
```

## 下载文件并保存
```java
String url = "http://127.0.0.1:8888" + "/exportZipFile";
// ================================
TimeInterval timer = DateUtil.timer();
// ------------------------------
// 1.下载文件(Stream)
InputStream inputStream = HttpRequest.get(url)
        .header("Authorization", token)
        .header("User-Agent", "Hutool.Http")
        // .form(Dict.create().set("p", "2").set("s", 20))
        .execute().bodyStream();

// -------------------------
String filePath = String.format("D:\\temppp\\tempp-%s-sss.zip", System.currentTimeMillis());
FileOutputStream outputStream = new FileOutputStream(filePath);
BufferedOutputStream bos = new BufferedOutputStream(outputStream);

int len;
byte[] bs = new byte[1024];
while ((len = inputStream.read(bs)) != -1) {
    Console.log("len= {}", len);
    bos.write(bs, 0, len);
}
bos.flush();
bos.close();
Console.log("文件保存路径: {}", filePath);
Console.log("耗时(毫秒): {} (ms)", timer.interval());

// ================================
// 2.下载文件(HttpUtil)
File downloadFile = cn.hutool.http.HttpUtil.downloadFileFromUrl("http://test.bbdun.cn/obpm/v2/file/wallhaven-p8lgvm.jpg", "/home/czz/file/xx.jpg");
log.info("exist={}, size={}", downloadFile.exists(), downloadFile.length());
```

---------------------
[hutool http文档](https://hutool.cn/docs/index.html#/http/%E6%A6%82%E8%BF%B0)
