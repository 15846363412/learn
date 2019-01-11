# 1.spingcloud--Eureka #
# 2什么是Eureka #
与zookpeer功能相近，都是对服务进行管理。简单的来说就是相当于一个中介，服务的提供这把服务暴露给eureka，服务的消费者到eureka这里拉取服务列表。eureka就起到了服务地址管理的作用。
# 3举个例子 #
当没有网约车之前，人们想要打车都需要去叫出租车。人们在路上是不能随表去叫停私家车的，私家车想要拉你但是又不知道你何时又需求，你又不敢做私家车，怕是黑车。当有了滴滴打车这样的平台之后，司机车主把自己的身份信息，车辆信息等注册到滴滴平台，而用户只需要到滴滴app上输入自己的需求然后就会匹配到车辆。那滴滴公司就相当于注册中心的作用。。。。

# 4eureka的原理 #
![](https://i.imgur.com/KdExWFM.png)

eurekaServer与服务的提供者之间是通过心跳进行监控的。当服务的提供者超过90ms没有给eureka发送心跳renew（相当于续约），注册中心认为其已经宕机，定期将其清理初服务列表。

- Eureka：就是服务注册中心（可以是一个集群），对外暴露自己的地址
- 提供者：启动后向Eureka注册自己信息（地址，提供什么服务）
- 消费者：向Eureka订阅服务，Eureka会将对应服务的所有提供者地址列表发送给消费者，并且定期更新
- 心跳(续约)：提供者定期通过http方式向Eureka刷新自己的状态

# 5入门小demo #
- 在idea中建立maven父工程cloud-domo打包方式pom
- 在pom文件中添加依赖


	 <groupId>cn.zyl.demo</groupId>
	    <artifactId>cloud-demo</artifactId>
	    <version>1.0.0-SNAPSHOT</version>
	    <modules>
	        <module>user-service</module>
	        <module>order-service</module>
	        <module>eureka-server</module>
	    </modules>
		<groupId>cn.zyl.demo</groupId>
		    <artifactId>cloud-demo</artifactId>
		    <version>1.0.0-SNAPSHOT</version>
		    <modules>
		        <module>user-service</module>
		        <module>order-service</module>
		        <module>eureka-server</module>
		    </modules>
	    <!--设置打包方式-->
	    <packaging>pom</packaging>
	    <parent>
	        <groupId>org.springframework.boot</groupId>
	        <artifactId>spring-boot-starter-parent</artifactId>
	        <version>2.1.1.RELEASE</version>
	    </parent>
	
	    <!--版本号统一管理-->
	    <properties>
	        <java.verion>1.8</java.verion>
	        <spring-cloud.version>Finchley.SR2</spring-cloud.version>
	    </properties>
	
	    <!--spring-cloud 版本，后面子项目直接拿坐标-->
	    <dependencyManagement>
	        <dependencies>
	            <dependency>
	                <groupId>org.springframework.cloud</groupId>
	                <artifactId>spring-cloud-dependencies</artifactId>
	                <version>${spring-cloud.version}</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	        </dependencies>
	    </dependencyManagement>

- 创建eureka-server子工程
- 导入依赖


		<parent>
		        <artifactId>cloud-demo</artifactId>
		        <groupId>cn.zyl.demo</groupId>
		        <version>1.0.0-SNAPSHOT</version>
		    </parent>
	    <modelVersion>4.0.0</modelVersion>
	
	    <artifactId>eureka-server</artifactId>
	
	    <dependencies>
	        <!-- eureka-server -->
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
	        </dependency>
	
	    </dependencies>
- 编写启动类


		**
		 * @author :zyl
		 * description :注册 中心
		 * date :2019/1/10
		 */
		@SpringBootApplication
		@EnableEurekaServer //启动Eureka-Server
		public class EurekaServerApplication {
		    public static void main(String[] args) {
		        SpringApplication.run(EurekaServerApplication.class);
		    }
		}

创建application.yml文件

	server:
	  port: 8761
	eureka:
	  instance:
	    hostname: 127.0.0.1
	  client:
	#    register-with-eureka: false
	#    fetch-registry: false
	    serviceUrl:
	      defautZone: http://${eureka.instance.hostname}:${server.port}/eureka
	spring:
	  application:
	    name: eureka-server


----------

- 创建服务提供者user-service项目
- 添加依赖
	 <parent>
	        <artifactId>cloud-demo</artifactId>
	        <groupId>cn.zyl.demo</groupId>
	        <version>1.0.0-SNAPSHOT</version>
	    </parent>
	    <modelVersion>4.0.0</modelVersion>
	
	    <artifactId>user-service</artifactId>
	
	    <dependencies>
	        <!-- spring-boot Web -->
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-web</artifactId>
	        </dependency>
	
	        <!-- eureka-client -->
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
	        </dependency>
	
	    </dependencies>

创建application.yml配置文件

		server:
		  port: 8771
		  #服务名称
		spring:
		  application:
		    name: user-service
		#注册中心
		eureka:
		  client:
		    service-url:
		      defaultZone: http://127.0.0.1:8761/eureka


服务提供

	**
	 * @author :zyl
	 * description :实现类
	 * date :2019/1/10
	 */
	@Service
	public class UserServiceImpl implements UserService {

    @Value("${server.port}")
    private String port;
    static  HashMap<Integer, User> userMap = new HashMap<>();
    static {
        User u1 = new User(1,"张三",11);
        User u2 = new User(2,"lis",10);
        User u3 = new User(3,"wf",20);
        userMap.put(u1.getId(), u1);
        userMap.put(u2.getId(), u2);
        userMap.put(u3.getId(), u3);
    }




    @Override
    public User findById(int id) {
        User user = userMap.get(id);
        user.setPort(port);
        return user;

    }

服务消费者

	/**
	 * @author :zyl
	 * description :
	 * date :2019/1/10
	 */
	@Service
	public class OrderServiceImpl implements OrderService {
    @Autowired
    private  RestTemplate restTemplate;

    @Autowired
    private DiscoveryClient discoveryClient;

    @Override
    public Order findById(int userId) {
        //通过serviceId 拉取服务列表
        List<ServiceInstance> instances = discoveryClient.getInstances("user-service");
        ServiceInstance instance = instances.get(0);
        //getForObject 第一个参数url代表访问路径 第二个参数代表 返回值类型
        String jsonStr = restTemplate.getForObject("http://"+instance.getHost()+":"+instance.getPort()+"/api/v1/user/2", String.class);
	//         String jsonStr = restTemplate.getForObject("http://127.0.0.1:8771/api/v1/user/2", String.class);

        JsonNode jsonNode = JsonUtils.str2JsonNode(jsonStr);
        Order order = new Order();
        jsonStr = "我是一个订单";
        order.setOrderName(jsonStr);
        order.setSerialId(UUID.randomUUID().toString());
        order.setUserName(jsonNode.get("data").get("name").textValue());
        order.setPort(jsonNode.get("data").get("port").textValue());
        return order;
    }

# 6.eureka的高可用 # 

模拟多个eureka-server相互注册

修改eureka-server配置文件application.yml
	server:
	  port: 8761
	
	eureka:
	  instance:
	    hostname: 127.0.0.1
	  client:
	    #自己不注册自己
	#    registerWithEureka: false
	    #不需要检索服务信息
	#    fetchRegistry: false
	    serviceUrl:
	      defaultZone: http://127.0.0.1:8762/eureka
	
	spring:
	  application:
	    name: eureka-server

复制一个eureka-server，方法同上

![](https://i.imgur.com/D095Mze.png)

此配置表示，端口号为8761的eureka会注册到8762上，端口号为8762的eureka会注册到8761上，因为eureka会把自己的注册列表复制更新到其他的注册中心上，所以我们最后访问 http://127.0.0.1:8761/eureka 和http://127.0.0.1:8762/eureka我们将看到这样的结果

![](https://i.imgur.com/kZSrtes.png)

# 7.服务的调用方和服务的提供方超时配置 #

一个服务既可以作为服务的提供方，也可以作为服务的调用方

一个服务的提供方（比如：user-service），在服务一启动时会检查eureka.client.register-with-erueka=true属性是否为true（不配置默认是true），如果是true就会把自己注册到注册中心（eureka-server）去，反之，如果为false就不会注册自己。

在服务注册完毕过后，服务的提供方会跟注册中心维持一个心跳（没间隔一段时间向注册中心发送一次rest请求）来告诉注册中心“我还活着”，这里我们叫服务的续约（renew），如果超过一段时间（默认是90秒）注册中心还没有收到心跳注册中心就会认为此服务已经宕机，然后把这个服务从注册列表中剔除。

配置如下

	eureka:
	  instance:
	     #服务的续约时间间隔
	     lease-renewal-interval-in-seconds: 30
	     #服务的失效时间间隔
	    lease-expiration-duration-in-seconds: 90

如果在开发阶段我们可以适当把这个值跳小一点，在生产环境中这个值尽量不用修改

服务的调用方（比如：order-service），在启动时候会检测eureka.client.fetch-registry=true属性是否为true（不配置同样是true）,如果是true，就会把注册中心（比如：eureka-server）的服务列表拉到本地缓存，并且没间隔一段时间（默认30秒）会重新拉取更新一次，每次调用都是从自己缓存中获取注册列表，后期如果注册中心挂了，服务依旧可以调用，但是不会更新注册列表了，所以注册中心尽量集群搭建，确保服务的可用性。

	eureka:
	  client:
	     #拉去列表时间间隔
	    registry-fetch-interval-seconds: 30
同样，生成环境我们不用修改，开发环境我们可以修改小一点。

# 8、失效剔除和自我保护 #

注册中心（eureka-server）会每间隔一段时间（默认60秒）把失效的服务（90秒没有收到心跳）从注册列表中剔除，也就是说一个服务，如果注册中心90秒没有收到心跳回复，不会立即剔除，而是每间隔一段时间统一剔除。

注册中心（eureka-server）会统计最近15分钟每分钟续约量是否超过85%，如果低于85%，eureka就会认为你没有宕机，可能是因为网络延迟等其他原因，eureka就会开启保护机制，这些实例就会被eureka保护起来，就算没有续约也不会剔除。我们在开发过程中很容易就满足续约量低于85%，所以一般开发时候会关闭自我保护（true：开启，false：关闭）

	eureka:
	  server:
	    # 扫描失效服务的间隔时间（缺省为60*1000ms）
	    eviction-interval-timer-in-ms: 60000 
	    # 关闭自我保护模式（缺省为打开）
	    enable-self-preservation: true 