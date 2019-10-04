

# 配置generatorConfig.xml文件

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
	<!-- 指定驱动物理地址 -->
	<classPathEntry    location="C:\Users\Administrator\.m2\repository\com\oracle\ojdbc6\6\ojdbc6-6.jar"/>
    <context id="testTables" targetRuntime="MyBatis3">
        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--mysql数据库连接的信息：驱动类、连接地址、用户名、密码 -->
        <jdbcConnection driverClass="oracle.jdbc.OracleDriver"
                        connectionURL="jdbc:oracle:thin:@192.168.10.14:1521/orcl" userId="shgl"
                        password="shgl">
        </jdbcConnection>

        <!--oracle配置-->
        <!-- <jdbcConnection driverClass="oracle.jdbc.OracleDriver" connectionURL="jdbc:oracle:thin:@127.0.0.1:1521:yycg"
            userId="yycg"
            password="yycg">
        </jdbcConnection> -->

        <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，
        为 true时把JDBC DECIMAL和NUMERIC类型解析为java.math.BigDecimal -->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="true"/>
        </javaTypeResolver>

        <!-- targetProject:生成model类的位置，重要！！ -->
        <javaModelGenerator targetPackage="com.ssyt.orderappeal.model" targetProject="src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false"/>
            <!-- 从数据库返回的值被清理前后的空格 -->
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!-- targetProject:mapper映射xml文件生成的位置，重要！！ -->
        <sqlMapGenerator targetPackage="com.ssyt.orderappeal.mapping"
                         targetProject="src/main/java">
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>

        <!-- targetPackage：mapper接口生成的位置，重要！！ -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.ssyt.orderappeal.dao"
                             targetProject="src/main/java">
            <property name="enableSubPackages" value="false"/>
        </javaClientGenerator>

        <!-- 指定数据库表，要生成哪些表，就写哪些表，要和数据库中对应，不能写错！ -->
        <table tableName="SHGL_ORDER_APPEAL_LOG"></table>
<!--    <table tableName="orders"></table>
        <table tableName="orderdetail"></table>
        <table tableName="user"></table> -->
    </context>
</generatorConfiguration>
```

# Maven插件形式

## 在maven配置文件中加入

```
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.2</version>
    <configuration>
        <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
        <verbose>true</verbose>
        <overwrite>true</overwrite>
    </configuration>
</plugin>
```

## 通过maven命令启动

右键run as通过maven运行 在 goals中 输入命令
```
mybatis-generator:generate
```

# Java类 执行

```

import org.junit.Before;
import org.junit.Test;
import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.internal.DefaultShellCallback;
import org.springframework.util.ResourceUtils;

import java.io.File;
import java.io.FileNotFoundException;
import java.util.ArrayList;
import java.util.List;

public class GeneratorTest {
    private File configFile;

    private String filename = "generator";

    @Before
    public void before() {
        //读取mybatis参数
        try {
			configFile = ResourceUtils.getFile(ResourceUtils.CLASSPATH_URL_PREFIX + "generator/" + filename + ".xml");
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		}
    }

    @Test
    public void test() throws Exception {
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
    }
}

```
