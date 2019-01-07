
----------
# 什么是Spring Boot #
Spring Boot简化了基于Spring的应用开发，通过少量的代码就能创建一个独立的、产品级别的Spring应用。 Spring Boot为Spring平台及第三方库提供开箱即用的设置，这样你就可以有条不紊地开始。多数Spring Boot应用只需要很少的Spring配置。

Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。用我的话来理解，就是Spring Boot其实不是什么新的框架，它默认配置了很多框架的使用方式，就像maven整合了所有的jar包，Spring Boot整合了所有的框架（不知道这样比喻是否合适）。

Spring Boot的核心思想就是约定大于配置，一切自动完成。采用Spring Boot可以大大的简化你的开发模式，所有你想集成的常用框架，它都有对应的组件支持。

# 什么是Spring Cloud #
Spring Cloud是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。Spring并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。

微服务是可以独立部署、水平扩展、独立访问（或者有独立的数据库）的服务单元，Spring Cloud就是这些微服务的大管家，采用了微服务这种架构之后，项目的数量会非常多，Spring Cloud做为大管家就需要提供各种方案来维护整个生态。

Spring Cloud就是一套分布式服务治理的框架，既然它是一套服务治理的框架，那么它本身不会提供具体功能性的操作，更专注于服务之间的通讯、熔断、监控等。因此就需要很多的组件来支持一套功能
# Spring Boot和Spring Cloud的关系 #
Spring Boot 是 Spring 的一套快速配置脚手架，可以基于Spring Boot 快速开发单个微服务，Spring Cloud是一个基于Spring Boot实现的云应用开发工具；Spring Boot专注于快速、方便集成的单个微服务个体，Spring Cloud关注全局的服务治理框架；Spring Boot使用了默认大于配置的理念，很多集成方案已经帮你选择好了，能不配置就不配置，Spring Cloud很大的一部分是基于Spring Boot来实现，可以不基于Spring Boot吗？不可以。

Spring Boot可以离开Spring Cloud独立使用开发项目，但是Spring Cloud离不开Spring Boot，属于依赖的关系。

Spring -> Spring Boot > Spring Cloud 这样的关系。


# hystrix #
## 1.1.简介 ##
Hystix，即熔断器。
![](https://i.imgur.com/fqgR8dG.png)

Hystix是Netflix开源的一个延迟和容错库，用于隔离访问远程服务、第三方库，防止出现级联失败。
![](https://i.imgur.com/q1FJQ1c.png)
## 1.2.熔断器的工作机制： ##
![](https://i.imgur.com/C5xHFsG.png)

正常工作的情况下，客户端请求调用服务API接口：
![](https://i.imgur.com/DJ5baSJ.png)

当有服务出现异常时，直接进行失败回滚，服务降级处理：
![](https://i.imgur.com/x06R4ag.png)

当服务繁忙时，如果服务出现异常，不是粗暴的直接报错，而是返回一个友好的提示，虽然拒绝了用户的访问，但是会返回一个结果。

这就好比去买鱼，平常超市买鱼会额外赠送杀鱼的服务。等到逢年过节，超时繁忙时，可能就不提供杀鱼服务了，这就是服务的降级。

系统特别繁忙时，一些次要服务暂时中断，优先保证主要服务的畅通，一切资源优先让给主要服务来使用，在双十一、618时，京东天猫都会采用这样的策略。

# 1.3雪崩问题 #
微服务中，服务间调用关系错综复杂，一个请求，可能需要滴啊用多个微服务接口才能实现，会形成非常复杂的调用链路：一次业务请求，需要调用多个服务，这些服务可能调用其他服务，如果此时，某个服务出现异常：请求阻塞，用户不会得到相应，tomcat的这个线程不会释放，于是越来越多的请求到来，越来越多的请求阻塞。

### Hystrix解决雪崩问题的手段： ###
#### 1.线程隔离：hystrix为每个依赖服务调用分配一个小的线程池，如果线程池已经满了调用被立即拒绝，默认不采用排队，加速失败判定时间####
#### 2.服务降级: 优先保证核心服务，而非核心服务不可用或者弱可用####

触发hystrix服务降级的情况：

- 线程池已满
- 请求超时


引入依赖

    <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>

Hystix的超时时间默认也是1000ms
我们可以通过hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds来设置Hystrix超时时间。

    hystrix:
    command:
  	  default:
        execution:
          isolation:
            thread:
              timeoutInMillisecond: 6000 # 设置hystrix的超时时间为6000ms


# hystrix小demo #
## 1、启动服务注册中心 ##
启动服务注册中心spring-cloud-discovery-eureka
## 2、修改服务提供者 ##
创建服务提供者spring-cloud-provider

a、在Controller中增加一个方法sayHello，代码如下：


    @GetMapping("/sayHello")
   	public String sayHello() {
		return "Hello this is provider";
	}

b、运行maven命令install，将项目jar包放到maven库中，并运行两个服务提供者，方法如下：

使用命令行窗口进入jar包所在的路径 执行命令：


    java -jar spring-cloud-provider-0.0.1-SNAPSHOT.jar --server.port=8081

再启动一个命令行窗口进入jar包所在的路径执行命令：

    java -jar spring-cloud-provider-0.0.1-SNAPSHOT.jar --server.port=8082

这样就启动了两个服务提供者，端口号分别问8081和8082。 
3、修改服务消费者
创建服务消费者spring-cloud-consumer，参照：Spring Cloud 服务消费者

a、在POM文件中加入依赖spring-cloud-starter-hystrix

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>

b、在spring boot 启动文件增加注解@EnableCircuitBreaker，开启熔断器功能


    @EnableCircuitBreaker
	@SpringBootApplication
	@EnableDiscoveryClient
	public class SpringCloudConsumerApplication {

    /**
     * 实例化RestTemplate，通过@LoadBalanced注解开启均衡负载能力.
     * @return restTemplate
     */
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConsumerApplication.class, args);
    }
	}

c、在ConsumerService中增加sayHello方法和回掉方法sayHelloFallBack，使用注解@HystrixCommand(fallbackMethod = “sayHelloFallBack”)指定回调函数，回调函数的返回值和参数必须和正式方法相同，不然会报错

	@HystrixCommand(fallbackMethod = "sayHelloFallBack")
	public String sayHello() {
	return this.restTemplate.getForObject("http://spring-cloud-provider/sayHello", String.class);
	}

	public String sayHelloFallBack() {
	return "error";
	}

d、在ConsumerController中增加方法调用ConsumerService中的sayHello方法
	
	@RestController 
	public class ConsumerController {
	
	@Autowired
	private ConsumerService consumerService;
	
	@GetMapping("/sayHello")
	public String sayHello() {
	return consumerService.sayHello();
	}

e、启动该工程，服务注册中心显示该服务信息
 
## 4、测试 ##
访问服务消费者的方法：http://localhost:8010/sayHello，返回：Hello this is provider信息。 
关闭服务提供者8082，再访问该路径，返回error信息。 
这说明Hystrix 已经生效了。