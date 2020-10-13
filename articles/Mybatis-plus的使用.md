### Mybatis-plus的使用

#### 代码自动生成

  1. 依赖坐标

     ```pro
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
     </dependency>
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-logging</artifactId>
     </dependency>
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-test</artifactId>
         <scope>test</scope>
     </dependency>
     <dependency>
         <groupId>org.projectlombok</groupId>
         <artifactId>lombok</artifactId>
         <version>1.18.12</version>
         <scope>provided</scope>
     </dependency>
     <dependency>
         <groupId>com.baomidou</groupId>
         <artifactId>mybatis-plus-boot-starter</artifactId>
         <version>3.4.0</version>
     </dependency>
     <dependency>
         <groupId>com.baomidou</groupId>
         <artifactId>mybatis-plus-generator</artifactId>
         <version>3.4.0</version>
     </dependency>
     <dependency>
         <groupId>org.apache.velocity</groupId>
         <artifactId>velocity-engine-core</artifactId>
         <version>2.2</version>
     </dependency>
     <dependency>
         <groupId>com.spring4all</groupId>
         <artifactId>swagger-spring-boot-starter</artifactId>
         <version>1.9.0.RELEASE</version>
     </dependency>
     ```

  2. 配置mysql的配置文件

      ```yaml
     spring:
       datasource:
         driverClassName: com.mysql.cj.jdbc.Driver
         url: jdbc:mysql://192.168.0.101:3306/envision_product?useUnicode=true&characterEncoding=utf-8
         username: root
         password: 123456root
     ```

3. 数据库中有要自动生成代码的表结构,表的注释和字段的注释一定要加,方便代码生成

4. MyBatis-Plus的配置如果有不要忘记配置,比如:

   ```java
   @Configuration
   public class MyBatisPlusConfig {
       //乐观锁插件配置
       @Bean
       public OptimisticLockerInnerInterceptor optimisticLockerInterceptor() {
           return new OptimisticLockerInnerInterceptor();
       }
       //分页插件配置
       @Bean
       public PaginationInnerInterceptor paginationInterceptor() {
           return new PaginationInnerInterceptor();
       }
   }
   ```

6. 代码自动生成器

   ```java
   //代码自动生成
   public class GCodeAuto {

       public static void main(String[] args) {
           //需要构建一个代码自动生成器->对象
           AutoGenerator mpg = new AutoGenerator();
           //配置策略
   
           //1.全局配置
           GlobalConfig gc = new GlobalConfig();
           //获取项目目录
           String projectPath = System.getProperty("user.dir");
           gc.setOutputDir(projectPath + "/product_service/src/main/java");
           gc.setAuthor("YeTing");
           gc.setOpen(false);
           gc.setFileOverride(true);//是否覆盖
           gc.setServiceName("%sService"); //去Service的I前缀
           gc.setIdType(IdType.ASSIGN_ID);
           gc.setDateType(DateType.ONLY_DATE);
           mpg.setGlobalConfig(gc);
   
           //2.设置数据源
           DataSourceConfig dsc = new DataSourceConfig();
           dsc.setUrl("jdbc:mysql://localhost:3306/envision_product?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8");
           dsc.setDriverName("com.mysql.cj.jdbc.Driver");
           dsc.setUsername("root");
           dsc.setPassword("123456root");
           dsc.setDbType(DbType.MYSQL);
           mpg.setDataSource(dsc);
   
           //3.包的配置
           PackageConfig pc = new PackageConfig();
           pc.setModuleName("product");
           pc.setParent("com.envision");
           pc.setEntity("entity");
           pc.setMapper("mapper");
           pc.setService("service");
           pc.setController("controller");
           mpg.setPackageInfo(pc);
   
           //4.策略配置
           StrategyConfig strategy = new StrategyConfig();
           strategy.setInclude("user");//设置要映射的表名
           strategy.setNaming(NamingStrategy.underline_to_camel);
           strategy.setColumnNaming(NamingStrategy.underline_to_camel);
           strategy.setEntityLombokModel(true); //自动lombok
           strategy.setRestControllerStyle(true);//restful的驼峰命名格式
           strategy.setControllerMappingHyphenStyle(true);//localhost:8080/hello_id_2
   //        //逻辑删除
   //        strategy.setLogicDeleteFieldName("deleted");
   //        //自动填充配置
   //        strategy.setTableFillList(Lists.list(
   //                new TableFill("create_time", FieldFill.INSERT),
   //                new TableFill("update_time", FieldFill.INSERT_UPDATE)
   //        ));
   //        //乐观锁配置
   //        strategy.setVersionFieldName("version");
   
           mpg.setStrategy(strategy);
   
           //执行
           mpg.execute();
       }
   }
   ```
   
   