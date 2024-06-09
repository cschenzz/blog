## import
```java
import cn.hutool.core.img.ImgUtil;
import cn.hutool.core.io.FileUtil;
import cn.hutool.core.io.IoUtil;

import java.io.BufferedOutputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.FileInputStream;
import java.io.BufferedInputStream;

import java.io.File;
import java.io.IOException;
```

## 读取stream中文本内容
```java
// 读取stream中文本内容
// InputStream inputStream = StreamUtils.nonClosing(inputMessage.getBody());
String text = IoUtil.readUtf8(inputStream);
```

## 读取文本文件内容
```java
String text = FileUtil.readUtf8String("D:\\temp\\json\\test.json");
```

## 在文本文件末尾追加内容
```java
// 文件不存在会新建
File file = FileUtil.appendUtf8String("txt content", "D:\\temp\\log\\test.log");
```

## File转到InputStream
```java
// FileInputStream
File file = FileUtil.file("D:\\temp\\log\\test.log");
FileInputStream inputStream = IoUtil.toStream(file);

// BufferedInputStream
BufferedInputStream bufferedInputStream = FileUtil.getInputStream(file);

// BufferedImage
java.awt.image.BufferedImage image = ImgUtil.read(inputStream);
```

## ClassPath读取资源文件
```java
import cn.hutool.core.io.IoUtil;
import cn.hutool.core.io.FileUtil;
import org.springframework.core.io.ClassPathResource;
import java.io.InputStream;
import java.io.File;

// 读取项目/resources/json/t1.json文件
ClassPathResource resource = new ClassPathResource("/json/t1.json");
InputStream is = resource.getInputStream();

// IoUtil
String text = IoUtil.readUtf8(is);
log.info("json:\r\n{}", text);

// 以行的方式读取文件(utf-8编码的文本文件)
File file = resource.getFile();
List<String> lines = FileUtil.readLines(file, java.nio.charset.StandardCharsets.UTF_8);
```

---------------------
- [hutool IO流相关](https://hutool.cn/docs/index.html#/core/IO/%E6%A6%82%E8%BF%B0)