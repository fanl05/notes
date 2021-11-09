# Docker 部署 tf-Serving 及调用
## 拉取 tf-serving 镜像
```bash
docker pull tensorflow/serving
```
## 下载 tf-serving
主要使用其自带的模型: saved_model_half_plus_two_cpu
Location: /.../serving/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_cpu
```bash
git clone https://github.com/tensorflow/serving
```
SavedModel 格式
```
assets/
variables/
    variables.data-00000-of-00001
    variables.index
saved_model.pb|saved_model.pbtxt
```
- assets 是可选目录，存放预测时的辅助文档信息；
- variables 存放 tf.train.Saver 时保存的变量信息；
- saved_model.pb 存放 MetaGraphDef，存储训练预测模型的程序逻辑和 SignatureDef 用于标记预测时的输入和输出
## 启动容器
port 8500: gRPC, port 5501: HTTP
```bash
docker run -p 8500:8500 -p 8501:8501 --mount type=bind,source=/opt/module/serving/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_cpu,target=/models/half_plus_two -e MODEL_NAME=half_plus_two -t tensorflow/serving &
```
- source: 模型在本机上的位置目录
- target: /models 固定，名字最好与模型名一致，指模型在容器中的路径
- MODEL_NAME: 设置 Docker 中的环境变量，和 target 名字一致
## HTTP 调用 tf-serving
URL
```
http://110.42.187.113:8501/v1/models/half_plus_two:predict
```
Request
```JSON
{
    "instances": [
        2.0,
        2.0,
        11.0,
        8.0,
        10.0
    ]
}
```
Response
```JSON
{
    "predictions": [
        3.0,
        3.0,
        7.5,
        6.0,
        7.0
    ]
}
```
## gRPC 调用 tf-serving
### Java 调用
POM.xml
```xml
<dependency>
     <groupId>com.yesup.oss</groupId>
     <artifactId>tensorflow-client</artifactId>
     <version>1.4-2</version>
</dependency>
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
```
Java Client
```java
ManagedChannel channel = ManagedChannelBuilder.forAddress("110.42.187.113", 8500).usePlaintext().build();
 
// block 模式
PredictionServiceGrpc.PredictionServiceBlockingStub stub = PredictionServiceGrpc.newBlockingStub(channel);
 
Predict.PredictRequest.Builder predictRequestBuilder = Predict.PredictRequest.newBuilder();
 
Model.ModelSpec.Builder modelSpecBuilder = Model.ModelSpec.newBuilder();
 
// name 为启动容器时指定的 MODEL_NAME
modelSpecBuilder.setName("half_plus_two");
 
// signatureName 默认为 serving_default
modelSpecBuilder.setSignatureName("serving_default");
predictRequestBuilder.setModelSpec(modelSpecBuilder);
 
// 入参访问默认是最新版本，如果需要特定版本可以使用 tensorProtoBuilder.setVersionNumber 方法
TensorProto.Builder tensorProtoBuilder = TensorProto.newBuilder();
tensorProtoBuilder.setDtype(DataType.DT_FLOAT);
TensorShapeProto.Builder tensorShapeBuilder = TensorShapeProto.newBuilder();
 
// 设置入参的维度
tensorShapeBuilder.addDim(TensorShapeProto.Dim.newBuilder().setSize(2));
tensorProtoBuilder.setTensorShape(tensorShapeBuilder.build());
 
List<Float> list = new ArrayList<>();
list.add(1.0f);
list.add(5.0f);
tensorProtoBuilder.addAllFloatVal(list);
 
predictRequestBuilder.putInputs("x", tensorProtoBuilder.build());
 
Predict.PredictResponse predictResponse = stub.predict(predictRequestBuilder.build());
predictResponse.getOutputsMap().forEach((k, v) -> System.out.println(v.getFloatValList()));
```
[TensorFlow Serving with Docker](https://github.com/tensorflow/serving/blob/master/tensorflow_serving/g3doc/docker.md)