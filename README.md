# Mybatis-plus-CodeGenetator
Mybatis-pluse实现逆向工程
当我们的程序需要访问数据库进行增删查改操作时，我们往往需要编写service层的接口及其实现类，mapper层的接口以及xml文件，还有pojo层的实体类等，这些编写工作无疑是繁重且容易出错的。好在Mabatie-plus的代码生成器可以帮助我们很好的完成逆向工程，提高编程效率。

# 引入依赖

首先需要引入Mybatis-plus及其代码生成器的依赖，以及数据库驱动和freemaker模板引擎的驱动。

```xml
<!--  Mybatis-Puls     -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.2</version>
</dependency>

<!--   Mybatis-Puls-代码生成器     -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.1</version>
</dependency>

<!--    freemarker模版引擎    -->
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.31</version>
</dependency>

<!--   mysql     -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

# 读取控制台内容

我们需要通过控制台输入数据库中的表名

```java
public static String scanner(String tip){
    Scanner scanner = new Scanner(System.in);
    StringBuilder help = new StringBuilder();
    help.append("请输入" + tip + ": ");
    System.out.println(help.toString());
    if(scanner.hasNext()){
        String ipt = scanner.next();
        if(StringUtils.isNotBlank(ipt)){
            return ipt;
        }
    }
    throw new MybatisPlusException("请输入正确的" + tip + "!");
}
```

# 新建代码生成器

```java
import com.baomidou.mybatisplus.generator.AutoGenerator;

AutoGenerator mpg = new AutoGenerator();
```

# 全局配置

配置总的输出路径以及各个类的作者，时间格式等信息。

```java
// 全局配置
GlobalConfig gc = new GlobalConfig();
String projectPath = System.getProperty("user.dir");    // 当前项目路径
// 输出路径
gc.setOutputDir(projectPath + "/src/main/java");
// 作者
gc.setAuthor("ytflyhw");
// 打开输出目录
gc.setOpen(false);
// xml 开启 BaseResultMap(通用查询映射结果)
gc.setBaseResultMap(true);
// xml 开启 BaseColumnList(通用查询结果列)
gc.setBaseColumnList(true);
// 日期格式，采用 Date
gc.setDateType(DateType.ONLY_DATE);
// 将配置加入生成器
mpg.setGlobalConfig(gc);
```

BaseResultMap(通用查询映射结果)，即pojo包中实体类的属性名与数据库字段名的映射，反应在mapper.xml文件中就是

```xml
<!-- 通用查询映射结果 -->
<resultMap id="BaseResultMap" type="com.xxxx.seckill.pojo.User">
    <id column="id" property="id" />
    <result column="nickname" property="nickname" />
    <result column="password" property="password" />
    <result column="salt" property="salt" />
    <result column="head" property="head" />
    <result column="register_date" property="registerDate" />
    <result column="last_login_date" property="lastLoginDate" />
    <result column="login_count" property="loginCount" />
</resultMap>
```

BaseColumnList(通用查询结果列)，即包含了所有字段的通用的SQL语句，反应在mapper.xml文件中就是

```xml
<!-- 通用查询结果列 -->
<sql id="Base_Column_List">
    id, nickname, password, salt, head, register_date, last_login_date, login_count
</sql>
```



# 数据源配置

配置数据源

```java
// 数据源配置
DataSourceConfig dsc = new DataSourceConfig();
// 数据源地址
dsc.setUrl("jdbc:mysql://localhost:3306/seckill");
// 驱动包
dsc.setDriverName("com.mysql.cj.jdbc.Driver");
// 用户名及密码
dsc.setUsername("root");
dsc.setPassword("root1234");
// 将配置加入生成器
mpg.setDataSource(dsc);
```

# 包配置

配置需要生成哪些包继续相关.java文件

```java
// 包配置
PackageConfig pc = new PackageConfig();
// 生成的相应文件位置
pc.setParent("com.xxxx.seckill")
    .setEntity("pojo")
    .setMapper("mapper")
    .setService("service")
    .setServiceImpl("service.impl")
    .setController("controller");
// 将配置加入生成器
mpg.setPackageInfo(pc);
```

# 自定义配置

自定义mapper的文件名以及内容

```java
// 自定义配置
InjectionConfig cfg = new InjectionConfig() {
    @Override
    public void initMap() {}
};

// 模版引擎
String templatePath = "/templates/mapper.xml.ftl";

// 自定义输出配置
List<FileOutConfig> focList = new ArrayList<>();
// 自定义配置优先输出
focList.add(new FileOutConfig(templatePath) {
    @Override
    public String outputFile(TableInfo tableInfo) {
        // 自定义输出文件名， 如果 Entity 设置了前后缀，此处 xml 的名称也会跟着发生变化
        return projectPath + "/src/main/resources/mapper/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
    }
});
// 将配置加入生成器
cfg.setFileOutConfigList(focList);
mpg.setCfg(cfg);
```

# 模板配置

配置各个类生成时所使用的模板文件

```java
// 配置模版
TemplateConfig templateConfig = new TemplateConfig()
    .setEntity("templates/entity2.java")
    .setMapper("templates/mapper2.java")
    .setService("templates/service2.java")
    .setServiceImpl("templates/serviceImpl2.java")
    .setController("templates/controller2.java");
templateConfig.setXml(null);
// 将配置加入生成器
mpg.setTemplate(templateConfig);
```

# 策略配置

```java
// 策略配置
StrategyConfig strategy = new StrategyConfig();
strategy.setNaming(NamingStrategy.underline_to_camel);  // 数据库表映射到实体的命名策略（下划线转驼峰）
strategy.setColumnNaming(NamingStrategy.underline_to_camel); // 数据库表字段映射到实体的命名策略
// 使用 lombok 模型（自动添加相应注解）
strategy.setEntityLombokModel(true);
// 生成 @RestController 控制器
// strategy.setRestControllerStyle(true);
strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
strategy.setControllerMappingHyphenStyle(true);
// 表前缀
strategy.setTablePrefix("t_");
// 将配置加入生成器
mpg.setStrategy(strategy);
```

# 运行生成器

```java
// 导入模板引擎
mpg.setTemplateEngine(new FreemarkerTemplateEngine());
// 运行生成器
mpg.execute();
```
