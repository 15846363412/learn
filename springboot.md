# 一、springboot学习总结

官网：https://spring.io/projects/spring-boot#learn

Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".

We take an opinionated view of the Spring platform and third-party libraries so you can get started with minimum fuss. Most Spring Boot applications need very little Spring configuration.

##1.springboot的真谛

> springboot使开发容易，基于spring的应用程序你只要run就可以了
>
> 我们可以用spring平台和第三方库你就能够开始一个小的程序，大多数的springboot程序需要很少的spring配置

简言之----用很少的配置文件就可以快速搭建一个spring的应用程序。



## 2.Quick start

Bootstrap your application with [Spring Initializr](https://start.spring.io/).（俗称springboot的脚手架）



#3.快速开启spring example

create a file called `app.groovy`

~~~
@RestController
class ThisWillActuallyRun {

	@RequestMapping("/")
	String home() {
		"Hello World!"
	}

}
~~~

~~~
spring run app.groovy
~~~

打开localhost:8080`

出现

~~~
Hello World!
~~~

# 4.开发我的第一个springboot程序

>1.创建maven工程，在pom.xml文件中添加依赖
>
>```
><parent>
>		<groupId>org.springframework.boot</groupId>
>		<artifactId>spring-boot-starter-parent</artifactId>
>		<version>2.1.2.BUILD-SNAPSHOT</version>
>	</parent>
>```



>添加classpath Dependencies
>
>springboot提供了大量的starter（启动器），让你添加jar包到你的根路径下，我门需要什么就添加什么的启动器。例如添加web的启动器。spring-boot-starter-web dependencu
>
>```
><dependencies>
>	<dependency>
>		<groupId>org.springframework.boot</groupId>
>		<artifactId>spring-boot-starter-web</artifactId>
>	</dependency>
></dependencies>
>```

就可以快速构建web项目。

> 在src/main/java下创建Hello.java
>
> ```
> @RestController
> @EnableAutoConfiguration
> public class Hello {
> 
> 	@RequestMapping("/")
> 	String home() {
> 		return "Hello World!";
> 	}
> 
> 	public static void main(String[] args) {
> 		SpringApplication.run(Hello.class, args);
> 	}
> 
> }
> ```

> @RestController:  @Responsebody+@Controller
>
> 返回结果给请求者

> @EnableAutoConfiguration:开启自动配置

# 5.创建一个实行jar的执行器

在pom.xml中添加dependency spring-boot-maven-plugin

~~~
spring-boot-maven-plugin
~~~

```
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```



# 6.Springboot的启动器（starters）

> 名称以spring-boot-starter-*的格式   ，其他第三方的启动器格式是thirdpartyproject-spring-boot-starter
>
> 