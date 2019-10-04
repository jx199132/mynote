# 修改pom文件

```
<packaging>jar</packaging>
改成
<packaging>war</packaging>
```
引入
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
```


# 新建一个类继承SpringBootServletInitializer

```
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.support.SpringBootServletInitializer;

/**
 * war启动配置
 */
public class ServletInitializer extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Springboot11WarApplication.class);//Springboot11WarApplication是springboot启动类
    }
}
```


# springboot启动类

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Springboot11WarApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot11WarApplication.class, args);
	}
}
```

# Maven打包命令

```
clean package
```

# 发布tomcat
maven打包后生产的xxxxxxx.war如果想改成ROOT.war ， 如下设置finalName


```
<plugin>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-maven-plugin</artifactId>
	<configuration>
		<finalName>ROOT</finalName>
	</configuration>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```



# 相关异常
## resources下文件找不到
resources下建立了map文件夹，文件夹存放了很多地图的json数据，

```
File file = ResourceUtils.getFile(ResourceUtils.CLASSPATH_URL_PREFIX + "map/" + code + ".json");
String json = FileUtils.read(file.getPath());
```
读取文件的时候在eclipse测试环境可以读取到文件，但是打的包却找不到 文件，代码应该改成下面

```
ClassPathResource resource = new ClassPathResource("map/" + code + ".json");
InputStream inputStream = resource.getInputStream();
String json = new String(FileUtils.readInputStream(inputStream));
```


附上FileUtils代码


```
public static String read(String filePath) {
        File file = new File(filePath);
        if (!file.exists()) {
            logger.error(MessageFormat.format("[FileUtil]: File:{0} not exists!", filePath));
        }
        StringBuilder sb = new StringBuilder();
        BufferedReader reader = null;
        try {
            reader = new BufferedReader(new FileReader(file));
            String line = null;
            while ((line = reader.readLine()) != null) {
                sb.append(line);
            }
        } catch (Throwable e) {
            logger.error(MessageFormat.format("[FileUtil]: File:{0} read error!", filePath));
        } finally {
            if (reader != null) {
                try {
                    //关闭文件
                    reader.close();
                } catch (Throwable e) {
                    logger.error(MessageFormat.format("[FileUtil]: File:{0} close error!", filePath));
                }
            }
        }
        return sb.toString();
    }




public static byte[] readInputStream(InputStream inputStream) throws IOException {
        byte[] buffer = new byte[1024];
        int len = 0;
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        while ((len = inputStream.read(buffer)) != -1) {
            bos.write(buffer, 0, len);
        }
        bos.close();
        return bos.toByteArray();
    }
```

## 乱码
在eclipse中读取resources下的文件没有出现乱码，但是打的包出现了乱码（window平台）

需要启动的时候指定UTF-8编码

```
java -Dfile.encoding=utf-8
```
