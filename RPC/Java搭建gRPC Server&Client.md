# Java 搭建 gRPC Server 及 Client
## 创建 Maven 项目
POM.xml
添加 grpc 依赖及 protobuf 插件
```xml
<properties>
    <java.version>1.8</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <grpc.version>1.34.1</grpc.version>
    <protobuf.version>3.12.0</protobuf.version>
    <protoc.version>3.12.0</protoc.version>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-bom</artifactId>
            <version>${grpc.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-netty-shaded</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-protobuf</artifactId>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-stub</artifactId>
    </dependency>
    <dependency>
        <groupId>com.google.protobuf</groupId>
        <artifactId>protobuf-java-util</artifactId>
        <version>3.12.0</version>
    </dependency>
</dependencies>

<build>
    <extensions>
        <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.6.2</version>
        </extension>
    </extensions>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.6.1</version>
            <configuration>
                <protocArtifact>com.google.protobuf:protoc:${protoc.version}:exe:${os.detected.classifier}
                </protocArtifact>
                <pluginId>grpc-java</pluginId>
                <pluginArtifact>io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${os.detected.classifier}
                </pluginArtifact>
                <protoSourceRoot>src/main/resources/proto</protoSourceRoot>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>compile-custom</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```
## 创建 .proto 文件
在 resources\proto 目录下添加 add.proto
```
syntax = "proto3";

option java_package = "a.b.c";
option java_outer_classname = "AddServiceProto";
option java_multiple_files = true;

service AddService{
  rpc add(AddRequest) returns (AddReply){}
}

message AddRequest{
  int32 a = 1;
  int32 b = 2;
}

message AddReply{
  int32 res = 1;
}
```
## 生成代码
执行 maven install

在 target\generated-sources\protobuf\ 下生成 grpc-java 及 java

分别将其设置为 generated sources root
## 编写 Server
```java
public class AddServer extends AddServiceGrpc.AddServiceImplBase {

    public static void main(String[] args) throws IOException {

        ServerBuilder.forPort(9999).addService(new AddServer()).build().start();

        System.out.println("Server Start At 9999");

        // 使程序不会提前终止
        while (true) {

        }

    }

    @Override
    public void add(AddRequest request, StreamObserver<AddReply> responseObserver) {
        int res = myAdd(request.getA(), request.getB());
        responseObserver.onNext(AddReply.newBuilder().setRes(res).build());
        responseObserver.onCompleted();
    }

    private int myAdd(int a, int b) {
        return a + b;
    }

}
```
## 编写 Client
```java
public class AddClient {

    private final AddServiceGrpc.AddServiceBlockingStub stub;

    public static void main(String[] args) {

        AddClient addClient = new AddClient();
        AddReply reply = addClient.stub.add(AddRequest.newBuilder().setA(100).setB(200).build());

        System.out.println(reply.getRes());

    }

    public AddClient() {
        ManagedChannel channel = ManagedChannelBuilder.forAddress("127.0.0.1", 9999).usePlaintext().build();
        stub = AddServiceGrpc.newBlockingStub(channel);
    }

}
```