# quickdata
spring boot配置阿里的druid数据库连接池<br>
配置好后来到druid后台管理页面，查看状态<br>
卧槽，druid简直太牛逼了<br>
下面是druid的配置方法：<br>
1.首先要在pom文件导入druid的依赖：<br>
<!--引入druid数据源-->
```
        <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.8</version>
        </dependency>
```
2.在项目中的配置文件（这里使用的是yml格式的配置文件，注意：yml里格式问题很多，千万别错，否则项目启动可能会失败）中配置使用druid，其中很重要的是`type: com.alibaba.druid.pool.DruidDataSource`,这里表明你使用的是druid数据源，
不写的话则使用springboot默认为你配置的数据源：<br>
  ```spring:
    datasource:
      username: root
      password: 123
      url: jdbc:mysql://localhost:3306/jdbc
      driver-class-name: com.mysql.jdbc.Driver
      type: com.alibaba.druid.pool.DruidDataSource
      initialSize: 5
      minIdle: 5
      maxActive: 20
      maxWait: 60000
      timeBetweenEvictionRunsMillis: 60000
      minEvictableIdleTimeMillis: 300000
      validationQuery: SELECT 1 FROM DUAL
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
      poolPreparedStatements: true
      #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
      filters: stat,wall,log4j
      maxPoolPreparedStatementPerConnectionSize: 20
      useGlobalDataSourceStat: true
      connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500 
  
```
3.向容器中加入druid组件，springboot推荐使用全注解的方式，新建一个类<br>
@Configuration表明这是一个配置类，一定要加，不然springboot识别不到<br>
 @ConfigurationProperties(prefix = "spring.datasource")和@Bean将配置文件中的datasource的参数拿来并将组件加入到ioc容器中去，
 这一步也很关键。
```
import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
       return  new DruidDataSource();
    }

    //配置Druid的监控
    //1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams = new HashMap<>();

        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        initParams.put("allow","");//默认就是允许所有访问
//        initParams.put("deny","192.168.15.21");

        bean.setInitParameters(initParams);
        return bean;
    }


    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");

        bean.setInitParameters(initParams);

        bean.setUrlPatterns(Arrays.asList("/*"));

        return  bean;
    }
}
```
按照以上步骤即可使用druid,每当你调用sql时，都可以在后台管理页面看到具体的状态<br>
最后，下面是配置好的druid管理图片<br>
<img src="https://raw.githubusercontent.com/watermakers/images/master/restful-crud-img/15.PNG"/>
