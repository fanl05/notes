# List Conversion
## 场景
使用 mapstruct 转换对象 List，将 Stu 转换为 StuVo
## 环境准备
```java
public class Stu {

    private String name;
    private Integer age;

    public Stu(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```
```java
public class StuVo {

    private String name;
    private String phoneNum;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPhoneNum() {
        return phoneNum;
    }

    public void setPhoneNum(String phoneNum) {
        this.phoneNum = phoneNum;
    }

    @Override
    public String toString() {
        return "StuVo{" +
                "name='" + name + '\'' +
                ", phoneNum='" + phoneNum + '\'' +
                '}';
    }
}
```
## 问题解决
只需加入 list 转换的接口，会循环调用对象转换的接口
```java
@Mapper(unmappedTargetPolicy = ReportingPolicy.ERROR)
public interface StuConverterBasic {

    StuConverterBasic INSTANCE = Mappers.getMapper(StuConverterBasic.class);

    @Mappings({
            @Mapping(target = "name", source = "stu.name"),
            @Mapping(target = "phoneNum", constant = "12345")
    })
    StuVo convert2Vo(Stu stu);

    ArrayList<StuVo> convert2Vos(List<Stu> stus);

}
```
## 结果验证
```java
Stu stu1 = new Stu("a", 10);
Stu stu2 = new Stu("b", 11);
List<Stu> list = new ArrayList<>();
list.add(stu1);
list.add(stu2);
List<StuVo> stuVos = StuConverterBasic.INSTANCE.convert2Vos(list);
System.out.println(stuVos);
```