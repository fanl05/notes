# ON DUPLICATE KEY UPDATE
## 场景
传入对象列表，若唯一键不存在则插入，唯一键存在则更新。在 Mapper.xml 文件中 <foreach> 标签遍历集合，在 on duplicate key update 后无法获取集合的每一个 item
## 场景还原
建表 SQL
```sql
create table student
(
	id int auto_increment
		primary key,
	name varchar(30) null,
	age int null,
	major varchar(30) null,
	hometown varchar(30) null,
	constraint name
		unique (name, age)
);
```
Entity 对象
```java
public class Student implements Serializable {

    private String name;

    private int age;

    private String major;

    private String hometown;

    public Student(String name, int age, String major, String hometown) {
        this.name = name;
        this.age = age;
        this.major = major;
        this.hometown = hometown;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getMajor() {
        return major;
    }

    public void setMajor(String major) {
        this.major = major;
    }

    public String getHometown() {
        return hometown;
    }

    public void setHometown(String hometown) {
        this.hometown = hometown;
    }

    @Override
    public String toString() {
        return "Student{" +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", major='" + major + '\'' +
                ", hometown='" + hometown + '\'' +
                '}';
    }
}
```
DAO 接口
```java
int insertOnDuplicateKeyUpdate(@Param("students") List<Student> students);
```
## 问题解决
使用 values(属性名) 获取该循环对应 item 的属性值
StudentMapper.xml
```xml
<insert id="insertOnDuplicateKeyUpdate" parameterType="com.vip.mybatis.entity.Student">
        insert into student (name, age, major, hometown) values
        <foreach collection="students" index="index" item="student" open="" separator="," close="">
            (#{student.name}, #{student.age}, #{student.major}, #{student.hometown})
        </foreach>
        on duplicate key update major = values(major), hometown = values(hometown)
</insert>
```
### 结果验证
调用接口
```java
Student student = new Student("test", 1, "test", "test");
List<Student> students = new ArrayList<>();
students.add(student);

SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.SIMPLE, true);
StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);

int count = mapper.insertOnDuplicateKeyUpdate(students);
System.out.println(count);
sqlSession.close();
```