# Swagger简单使用教程

Swagger是一款API文档生成工具，支持多种语言，支持在线运行实例，毕设中使用到了Swagger，这里做个记录

### SpringMVC整合Swagger

#### 引入jar包

在pom.xml文件中添加以下依赖

```xml
<!-- swagger2 核心依赖 -->
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.6.1</version>
</dependency>
<!-- swagger-ui 为项目提供api展示及测试的界面 -->
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>2.6.1</version>
</dependency>
```



#### 添加Swagger配置

这里使用 JavaConfig 的方法配置Swagger, 配置类为 SwaggerConfig，该类采用注解的方式注入Spring中，需要放在Spring能够扫描到的包中

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 *
 * @author weigs
 * @date 2018/5/9 0009
 */

@Configuration
@EnableSwagger2
@EnableWebMvc
public class SwaggerConfig {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                .build()
                .apiInfo(apiInfo());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("毕业设计API")
                .description("提供API给前端")
                .termsOfServiceUrl("")
                .contact(new Contact("", "", ""))
                .license("")
                .licenseUrl("")
                .version("1.0.0")
                .build();
    }
}
```



#### Controller中加注解



```java
@Api(tags = {"我是分类名"}, description = "我是描述")
@RestController
@RequestMapping("/swaggertest")
public class TestController {

    @ApiOperation("我是接口")
    @RequestMapping(value = "/test", method = RequestMethod.GET)
    public void test(@ApiParam(name = "用户id") @RequestParam("uid") String uid) {

    }
}
```

Swagger中包含一些特定的注解，如下：

- `@API` 表示一个开放的API，可以通过description简要描述该API的功能，一般和 Spring 的 `@Controller`一起。
- 在一个`@API`下，可有多个`@ApiOperation`，表示针对该API的CRUD操作。在ApiOperation Annotation中可以通过value，notes描述该操作的作用，response描述正常情况下该请求的返回对象类型。
- 在一个ApiOperation下，可以通过ApiResponses描述该API操作可能出现的异常情况。
- `@ApiParam`用于描述该API操作接受的参数类型
- `@ApiModel`用来描述封装的参数对象与返回的参数对象
- `@ApiModelProperty`描述ApiModel的属性



#### 集成Swagger-ui界面

Swagger 提供了一个静态项目 Swagger-ui，可以帮忙将获得的JSON格式的API优美的展示出来并且帮助测试这些接口。 集成 Swagger-ui 很简单，从<https://github.com/swagger-api/swagger-ui> 获取其所有的 dist 目录下东西放到需要集成的项目里，比如这里放入 src/main/webapp/swagger/ 目录下。由于是静态文件，所以需要在SpringMVC.xml文件中配置能够访问静态文件，配置如下：在springmvc-servlet.xml中添加如下内容（如果已经添加，忽略此步骤）

```xml
<!--静态资源的默认配置-->
<mvc:default-servlet-handler/>
```

在`index.html` 中找到`url` ，更改为http://host:port/projectname/v2/api-docs，启动项目，访问http://host:port/projectname/swagger/index.html，将会出现以下界面，证明配置集成成功。





