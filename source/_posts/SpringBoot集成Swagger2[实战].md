---
title: SpringBoot集成Swagger2
---

在pom.xml文件中引入swagger2相关依赖包
```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>${springfox.swagger.version}</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>${springfox.swagger.version}</version>
</dependency>
```

```java
@EnableSwagger2
@Configuration
public class SwaggerConfig {

    @Bean(value = "authApi")
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .groupName("默认接口")
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example.resource"))//为当前包路径
                .paths(PathSelectors.any())
                .build();
    }

    @Bean(value = "patApi")
    public Docket createRestPatApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .groupName("达人拍拍接口")
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example.controller"))//为当前包路径
                .paths(PathSelectors.any())
                .build();
    }

    /**
     *构建api文档的详细信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot 使用 Swagger2 构建RESTful API")//页面标题
                .contact(new Contact("swagger2", "http://www.example.com/", "example@gmail.com"))//创建人
                .version("1.0")//版本号
                .description("API 描述")//描述
                .build();
    }
}
```