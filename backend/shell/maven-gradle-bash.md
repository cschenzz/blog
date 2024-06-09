## maven命令
```bash
mvn clean package -Dmaven.test.skip=true

#打生产环境包(跳过测试用例)
mvn clean package -Dmaven.test.skip=true -Pprod

#powerShell中执行
mvn clean package '-Dmaven.test.skip=true' -Pprod

mvn clean
mvn install
#install包括package过程
mvn install -Dmaven.test.skip=true
mvn package
mvn package -Dmaven.test.skip=true
mvn test

#运行springboot项目,停止ctrl+c
mvn spring-boot:run
#启动停止,一般用上面这条命令
mvn spring-boot:start
mvn spring-boot:stop
#--------------
#先在xx-security执行
mvn clean install

#再到xx-admin目录下，执行下面命令，就可以打成xx-admin.jar
mvn clean package -Dmaven.test.skip=true

#如果想打成war，则在xx-admin目录下，执行
mvn clean package -f pom-war.xml

#通过mvn dependency:tree查看依赖树,解决依赖jar冲突问题
mvn dependency:tree

# 查看slf4j的依赖关系(看它是通过那个dependency引入的), groupId: org.slf4j, artifactId: slf4j-simple
mvn dependency:tree -Dverbose -Dincludes=org.slf4j:slf4j-simple
#------------------
```

## gradle命令
```bash
#查看依赖
gradlew dependencies

#清理
gradlew clean

#打包
gradlew war
gradlew bootWar
gradlew bootJar

#run运行
gradlew bootRun

#java -jar方式运行
java -jar app.jar --name="Spring" --server.port=9999

#android studio中的使用

#列出gradle的task
gradlew tasks

#打包build
gradlew build

#打release包
gradlew assembleRelease

```