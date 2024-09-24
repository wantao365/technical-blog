参考来源：[xjar-国内](https://gitee.com/core-lib/xjar)       [xjar-国外](https://github.com/core-lib/xjar)       [xjar-maven-plugin](https://github.com/core-lib/xjar-maven-plugin)

Spring Boot JAR 安全加密运行工具，同时支持的原生JAR。

基于对JAR包内资源的加密以及拓展ClassLoader来构建的一套程序加密启动，动态解密运行的方案，避免源码泄露或反编译。

功能特性

- 无需侵入代码，只需要把编译好的JAR包通过工具加密即可。
- 完全内存解密，杜绝源码以及字节码泄露或反编译。
- 支持所有JDK内置加解密算法。
- 可选择需要加解密的字节码或其他资源文件，避免计算资源浪费。

# 【方式一不推荐】（自行加密）

## 1. 引入maven依赖

```xml
<dependency>
    <groupId>com.github.core-lib</groupId>
    <artifactId>xjar</artifactId>
    <version>v2.0.6</version>
</dependency>
 
<dependency>
   <groupId>com.github.core-lib</groupId>
   <artifactId>loadkit</artifactId>
   <version>v1.0.0</version>
 </dependency>

<!-- 可能会缺，本人缺 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-compress</artifactId>
    <version>1.18</version>
</dependency>
 
<!--这个配置在</dependencies> 之外-->
<!-- 设置 jitpack.io 仓库 -->
<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>
```

 但是如果使用idea的pom文件来下载上面两个依赖包，会一直不能成功下载，所以使用下面方法：

- [xjar](https://mvnrepository.com/artifact/com.github.core-lib/xjar/4.0.2) 然后点击页面的view all下载jar

- [loadkit](https://mvnrepository.com/artifact/com.github.core-lib/loadkit/v1.0.0) 然后点击页面的view all下载jar

- 上面两个包下载到本地之后，启动cmd，执行下面两段命令（默认我们都有配置maven的环境变量），注意下面命令中的文件路劲改成自己的下载的对应路劲

  > ```mvn
  > mvn install:install-file -Dfile=F:\GuGeDonwLoad\xjar-v2.0.6.jar  -DgroupId=com.github.core-lib -DartifactId=xjar -Dversion=v2.0.6 -Dpackaging=jar
  > ```
  >
  > ```mvn
  > mvn install:install-file -Dfile=F:\GuGeDonwLoad\loadkit-v1.0.0.jar   -DgroupId=com.github.core-lib -DartifactId=loadkit -Dversion=v1.0.0 -Dpackaging=jar
  > ```

  执行之后，已经把对应的jar包通过mvn下载到默认的仓库路劲了，如果我们idea-maven仓库下载路劲在别的目录，可以打开刚编译过的目录下，拷贝至自定义的目录下。这时pom文件会自动刷新，也就不会爆红了。

## 2.加密jar

在工程下随便创建一个类，加入下面的main方法，运行结束后会生成对应的加密jar包，然而你会发现加密后的jar包会比没有加密的大一倍，哈哈，我也不知道为啥。

```java
import io.xjar.XConstants;
import io.xjar.XKit;
import io.xjar.boot.XBoot;
import io.xjar.key.XKey;

/**
 * TODO
 * @author wantao
 * @version 1.0
 * @date 2022/3/24 14:41
 */
public class Test {
    public static void main(String[] args) {
        try {
            // 危险加密模式，即不需要输入密码即可启动的加密方式，这种方式META-INF/MANIFEST.MF中会保留密钥，请谨慎使用！
            String password = "io.xjar";
            XKey xKey = XKit.key(password);
            XBoot.encrypt("/path/to/read/plaintext.jar", "/path/to/save/encrypted.jar", xKey, XConstants.MODE_DANGER);

            // Spring-Boot Jar包解密
            String password = "io.xjar";
            XKey xKey = XKit.key(password);
            XBoot.decrypt("/path/to/read/encrypted.jar", "/path/to/save/decrypted.jar", xKey);
            
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

```java
XCryptos.encryption()
        .from("/Users/jerry/yl/springboot_webmagic/target/springboot_webmagic-0.0.1-SNAPSHOT.jar")        指定加密的jar包路径
        .use("zhaojun98xyz")			//指定加密密码
//				.include("/io/xjar/**/*.class")	//指定要加密的资源相对于classpath的ANT路径表达式
//				.include("/mapper/**/*Mapper.xml")//指定要加密的资源相对于classpath的正则路径表达式
//				.exclude("/static/**/*")		//指定不加密的资源相对于classpath的ANT路径表达式
//				.exclude("/conf/*")			//指定不加密的资源相对于classpath的正则路径表达式
				.to("/Users/jerry/fsdownload/webmagic_demo.jar");	//指定加密后JAR包输出路径, 并执行加密.
```

windows启动或者当前窗口启动方式：cmd 到加密的jar下，启动jia包```java -jar /path/to/encrypted.jar```，会提示输入密码，成功则启动。也可以```java -jar /path/to/encrypted.jar --xjar.password=PASSWORD```,但是容易密码泄露。

可能在启动加密后的jar会提示：Exception in thread "main" java.lang.ClassNotFoundException:，在pom加入下面代码，对应mainClass写自己的。

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
    	<mainClass>com.example.demo.DemoApplication</mainClass>
    </configuration>
</plugin>
```

linux后台启动，首先利用ieda创建一个properties文件，写入密码。然后放置到linux的某个目录下，运行即可

```xml
// 对于 nohup 或 javaw 这种后台启动方式，无法使用控制台来输入密码，推荐使用指定密钥文件的方式启动
nohup java -jar /path/to/encrypted.jar --xjar.keyfile=/path/to/xjar.key      > nohup.out 2>&1 & 
```

> xjar.key 
>
> passward:wantao
>
> 他的加密方式，应该是让xjar 使用自定义的类加载器，去将所有的类进行单独的类加载。
>
> 具体参数说明详见xjar作者介绍文档。

# 【方式二推荐】（通过maven插件）

## 1.引入maven依赖

```xml
<!-- 设置 jitpack.io 仓库 -->
<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>
<!-- 添加 XJar 依赖 -->
<dependencies>
    <dependency>
        <groupId>com.github.core-lib</groupId>
        <artifactId>xjar</artifactId>
        <version>4.0.2</version>
        <!-- <scope>test</scope> -->
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <!-- XJar 加密之后找不到main可添加 -->
            <configuration>
                <mainClass>com.example.demo.DemoApplication</mainClass>
            </configuration>
        </plugin>
        <plugin>
            <groupId>com.github.core-lib</groupId>
            <artifactId>xjar-maven-plugin</artifactId>
            <version>4.0.2</version>
            <executions>
                <execution>
                    <goals>
                        <goal>build</goal>
                    </goals>
                    <!--  <phase>none</phase> -->
                    <phase>package</phase>
                    <configuration>
                        <includes>
                            <include>com/example/**</include>
                        </includes>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## 2. 通过maven插件直接打包

```xml
package -Dxjar.password=123456 -Dmaven.test.skip=true
```

- 会生成***.xjar  xjar.go

- 安装golang环境([window自行安装](https://golang.google.cn/))

  ```shell
  yum install -y epel-release
  yum install golang
  go version
  ```

- 然后编译

  ```shell
  go build xjar.go
  ```

- 然后启动

  - ```
    linux: ./xjar java -jar ./demo-0.0.1-SNAPSHOT.xjar
    ```

  - ```
    windows: xjar.exe java -jar demo-0.0.1-SNAPSHOT.xjar
    ```

> 我使用了 jd，luyten xJad 三个工具， 正常反编译的jar 都可以得到代码， 用xjar 加密的 spring boot jar 都查看不了， 不知道以后会不会被破解，我把这三个工具都上传到我的百度云盘,有兴趣可以测试一下， 

