# SpringBoot 部署
1. 新建 Springboot App
2. 添加 Web 依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
3. 添加 spring-boot-maven-plugin 插件
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
4. 新增接口
```xml
@RequestMapping(value = "/sayHello",method = RequestMethod.GET)
public String sayHello(){
    return "Hello";
}
```
## 打 Jar 包部署
5. 执行 Maven package 或 `mvn clean package -Dmaven.test.skip=true` 在 target 目录下生成 jar 包
6. 执行 java -jar xxx.jar 启动应用
## Docker 部署
5. 编写 Dockerfile
```
FROM java:8
COPY *.jar /app.jar

CMD ["--server.port=8080"]

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]
```
6. 生成 image
```
docker built -t [镜像名] .
```
7. 启动 image
```bash
docker run -d -p 8080:8080
```