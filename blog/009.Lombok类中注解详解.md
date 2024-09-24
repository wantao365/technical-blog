1、@Data：在JavaBean中使用，注解包含包含getter、setter、NoArgsConstructor注解

　　@Value注解和@Data类似，区别在于它会把所有成员变量默认定义为private final修饰，并且不会生成set方法

2、@getter：在JavaBean中使用，注解会生成对应的getter方法

3、@setter：在JavaBean中使用，注解会生成对应的setter方法

4、@NoArgsConstructor：在JavaBean中使用，注解会生成对应的无参构造方法

5、@AllArgsConstructor：在JavaBean中使用，注解会生成对应的有参构造方法

　　@RequiredArgsConstructor ：生成private构造方法，使用staticName选项生成指定名称的static方法。

6、@ToString：在JavaBean中使用，注解会自动重写对应的toStirng方法

　　@ToString(exclude={"column1","column2"})：排除多个column列所对应的元素

　　@ToString(of={"column1","column2"})：只生成包含多个column列所对应的元素

7、@EqualsAndHashCode：在JavaBean中使用，注解会自动重写对应的equals方法和hashCode方法

8、@Slf4j：在需要打印日志的类中使用，项目中使用slf4j日志框架

9、@Log4j：在需要打印日志的类中使用，项目中使用log4j日志框架

10、@NonNull：注解快速判断是否为空,为空抛出java.lang.NullPointerException

11、@Synchronized：注解自动添加到同步机制，生成的代码并不是直接锁方法,而是锁代码块， 作用范围是方法上

12、@Cleanup：注解用于确保已分配的资源被释放（IO的连接关闭）

**重点线**

13、@Accessors(chain = true)：链式风格，在调用set方法时，返回这个类的实例对象

```java
@Accessors(chain = true)
@Setter
@Getter
public class Student {
    private String name;
    private int age;
}

/************************************/
public class Student {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public Student setName(String name) {
        this.name = name;
        return this;
    }

    public int getAge() {
        return age;
    }

    public Student setAge(int age) {
        this.age = age;
        return this;
    }
}

////////////////////////////////////////////////////////////
Student student = new Student()
        .setAge(24)
        .setName("zs");
```

14、@RequiredArgsContructor(staticName = "of")：生成一个静态方法，用于构建本类对象，与@NonNull联用，指定那些属性是本方法参数

```java
@Accessors(chain = true)
@Setter
@Getter
@RequiredArgsConstructor(staticName = "of")
public class Student {
    @NonNull 
    private String name;
    private int age;
}

/******************************************/

@Accessors(chain = true)
@Setter
@Getter
public class Student {
    private String name;
    private int age;

    public static Student of(String name) {
        return new Student().setName(name);
    } 
}


/////////////////////////////////////////////////////////////////
Student student = Student.of("zs");
```

15、@Builder：构建者模式

```java
@Builder
public class Student {
    private String name;
    private int age;
}

/****************************************/

@Getter
@Setter
public class Student {
    private String name;
    private int age;

    public static Builder builder(){
            return new Builder();
    }
    public static class Builder{
            private String name;
            private int age;
            public Builder name(String name){
                    this.name = name;
                    return this;
            }

            public Builder age(int age){
                    this.age = age;
                    return this;
            }

            public Student build(){
                    Student student = new Student();
                    student.setAge(age);
                    student.setName(name);
                    return student;
            }
    }
}


/////////////////////////////////////////////////////////////
Student student = Student.builder().name("zs").age(24).build();
```

16、@Delegate：代理模式

```java
@AllArgsConstructor
public abstract class FilterRestTemplate implements RestOperations {
    @Delegate
    protected volatile RestTemplate restTemplate;
}

/*********************************************/

public abstract class FilterRestTemplate implements RestOperations {

    protected volatile RestTemplate restTemplate;

    protected FilterRestTemplate(RestTemplate restTemplate) {
            this.restTemplate = restTemplate;
    }

    @Override
    public <T> T getForObject(String url, Class<T> responseType, Object... uriVariables) throws RestClientException {
            return restTemplate.getForObject(url,responseType,uriVariables);
    }

    @Override
    public <T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException {
            return restTemplate.getForObject(url,responseType,uriVariables);
    }

    @Override
    public <T> T getForObject(URI url, Class<T> responseType) throws RestClientException {
            return restTemplate.getForObject(url,responseType);
    }

    @Override
    public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Object... uriVariables) throws RestClientException {
            return restTemplate.getForEntity(url,responseType,uriVariables);
    }
    //其他实现代码略。。。
}
```

