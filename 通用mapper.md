#### 通过Mapper可以极大的方便开发人员。可以随意的按照自己的需要选择通用，极其方便的使用MyBatis单表的CRUD，支持单表操作，不支持通用的多表联合查询。 ####

通用Mapper
 

1、通用Mapper的使用
1.0、实体类


	     1 @Table(name = "tb_user")
	     2 public class User {
	     3 
	     4 
	     5 @Id
	     6 @GeneratedValue(generator = "JDBC")
	     7 private Long id;
	     8 // 用户名
	     9 private String userName;
	    10 // 密码
	    11 private String password;
	    12 // 姓名
	    13 private String name;
	    14 // 年龄
	    15 private Integer age;
	    16 // 性别，1男性，2女性
	    17 private Integer sex;
	    18 // 出生日期
	    19 private Date birthday;
	    20 // 创建时间
	    21 private Date created;
	    22 // 更新时间
	    23 private Date updated;
	    24//set、get方法....
	    25 
	    26 }
复制代码
 

泛型(实体类)<T>的类型必须符合要求
复制代码

     1 1、表名默认使用类名,驼峰转下划线(只对大写字母进行处理),如UserInfo默认对应的表名为user_info。
     2 
     3 2、表名可以使用@Table(name = "tableName")进行指定,对不符合第一条默认规则的可以通过这种方式指定表名.
     4 
     5 3、字段默认和@Column一样,都会作为表字段,表字段默认为Java对象的Field名字驼峰转下划线形式.
     6 
     7 4、可以使用@Column(name = "fieldName")指定不符合第3条规则的字段名
     8 
     9 5、使用@Transient注解可以忽略字段,添加该注解的字段不会作为表字段使用.
    10 
    11 6、建议一定是有一个@Id注解作为主键的字段,可以有多个@Id注解的字段作为联合主键.
    12 
    13 获取自增主键：
    14 
    15 //不限于@Id注解的字段,但是一个实体类中只能存在一个(继承关系中也只能存在一个)
    16 @GeneratedValue(generator = "JDBC")
    17 private Integer id;
    18 这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段）。
复制代码

1.1、导入依赖
1 <!-- 通用mapper -->
2 <dependency>
3 <groupId>tk.mybatis</groupId>
4 <artifactId>mapper-spring-boot-starter</artifactId>
5 <version>2.0.2</version>
6 </dependency>

1.2、开启包扫描
注意导入的包，一定要和导入的的依赖有关的包


	 1 import org.springframework.boot.SpringApplication;
	 2 import org.springframework.boot.autoconfigure.SpringBootApplication;
	 3 import tk.mybatis.spring.annotation.MapperScan;
	 4 
	 5 @SpringBootApplication
	 6 @MapperScan("com.myx.demo.mapper")
	 7 public class Application {
	 8 public static void main(String[] args) {
	 9 SpringApplication.run(Application.class);
	10 }
	11 }

 

1.3、UserMapper接口继承通用Mapper

	1 package com.myx.demo.mapper;
	2 import com.myx.demo.pojo.User;
	3 import tk.mybatis.mapper.common.Mapper;
	4 
	5 public interface UserMapper extends Mapper<User>{
	6 
	7 }

1.4、SpringBoot默认的文件名Application.yml配置

	 1 server:
	 2   port: 8080
	 3 
	 4 logging:
	 5   level:
	 6     com.myx.demo:
	 7       debug
	 8 spring:
	 9   datasource: #spring数据源的配置
	10     url: jdbc:mysql://localhost:3306/day11mybatis
	11     username: root
	12     password: 123

1.5、通用Mapper的测试类

	1 @RunWith(SpringRunner.class)
	2 @SpringBootTest(classes = Application.class)
	3 public class NewUserMapperTest {
	4 
	5     @Autowired
	6     private UserMapper userMapper;
	7 
	8 
	9 }

1.5.1 id 的专用查询方法
	1 @Test
	2     public void testSelectId() {
	3         userMapper.selectByPrimaryKey(1L);//id专用查询方法
	4         System.out.println( userMapper.selectByPrimaryKey(1L));
	5     }
	

1.5.2 属性查询

	1 @Test
	2     public void testSelectName() {
	3         User user = new User();
	4         user.setName("张三");//哪个属性有值where条件就根据哪个属性生成，id除外
	5         List<User> select = userMapper.select(user);
	6         System.out.println(select);
	7     }

1.5.3 属性值唯一查询

	1 @Test
	2     public void testSelectOne() {
	3         User user = new User();
	4         user.setUserName("zhangsan");//查询值有多个会报错
	5 
	6         System.out.println(userMapper.selectOne(user));
	7 
	8     }



1.5.4 统计个数
	1 @Test
	2     public void testSelectCount() {
	3         System.out.println(userMapper.selectCount(null));//没有查询条件就全查
	4     }


1.5.5 插入

	 1 @Test
	 2     public void testInsert() {
	 3         User user = new User();
	 4         user.setUserName("孙悟空");
	 5         user.setName("sunwukong");
	 6         user.setAge(500);
	 7         user.setId(50L);
	 8         user.setPassword("234");
	 9         int insert = userMapper.insert(user);
	10         System.out.println(insert);
	11     }


1.5.6 selective插入

	 1 /*
	 2     * insertSelective方法：有值才操作，没有值不理会，比上insert方法效率高
	 3     * */
	 4     @Test
	 5     public void testInsertSelective() {
	 6         User user = new User();
	 7         user.setUserName("猪八戒");
	 8         user.setName("sunwukong");
	 9         user.setAge(500);
	10         user.setId(51L);
	11         user.setPassword("234");
	12         user.setCreated(new Date());
	13         int insert = userMapper.insertSelective(user);
	14         System.out.println(insert);
	15     }



1.5.7 删除

	1  @Test
	2     public void testDelete() {
	3         User user = new User();
	4         user.setUserName("孙悟空");
	5         int delete = userMapper.delete(user);
	6         System.out.println(delete);
	7     }


 

 1.5.8 根据id删除，主键的特有方法

	 1  @Test 2 public void testDeleteByPrimaryKey() { 3 userMapper.deleteByPrimaryKey(51L); 4 } 
	 1.5.9 修改
	复制代码
	 1 /*
	 2     * 指定的属性改变为指定的值，其余的为null
	 3     * */
	 4     @Test
	 5     public void testUpdateByPrimaryKey() {
	 6         User user = new User();
	 7         user.setId(36L);
	 8         user.setUserName("李白");
	 9         user.setName("libai");
	10 
	11         userMapper.updateByPrimaryKey(user);
	12     }


1.5.10 selecttive修改

	    /*
	    * 只改变指定的属性，其余属性不变
	    * */
	    @Test
	    public void testUpdatePriimarySelective() {
	        User user = new User();
	        user.setId(33L);
	        user.setUserName("王昭君");
	        user.setName("wangzhaojun");
	        user.setUpdated(new Date());
	        userMapper.updateByPrimaryKeySelective(user);
	    }



2 通用Mapper分页
2.1 分页的依赖
	<!--分页的依赖-->
	        <dependency>
	            <groupId>com.github.pagehelper</groupId>
	            <artifactId>pagehelper-spring-boot-starter</artifactId>
	            <version>1.2.3</version>
	        </dependency>
2.2 分页方法的测试

	@RunWith(SpringRunner.class)
	@SpringBootTest(classes = Application.class)
	
	public class MapperPageQueryTest {
	    @Autowired
	    private UserMapper userMapper;
	
	    @Test
	    public void testPageQuery() {
	        //开启分页
	        PageHelper.startPage(3,5);
	        //不包含总的页数，总的元素个数
	        List<User> select = userMapper.select(null);
	        System.out.println(select);
	    }

	    @Test
	    public void testPageInfoQuery() {
	        //开启分页
	        PageHelper.startPage(3,5);
	        //用page类型或者pageInfo类型目的就是获取总页数以及总条数
	        Page<User> userPage = (Page<User>) userMapper.select(null);
	        //分页返回的结果
	        //PageInfo<User> pageInfo = new PageInfo<>(select);
	        System.out.println(userPage);
	    }
	}

3 通用Mapper组合查询和排序
3.1 添加普通条件

	/*
	    * example要表达的含义为？？？
	    * 组合的查询条件
	    * */
	    @Test
	    public void testSelectByExample(){
	        //条件的组合工具
	        Example example = new Example(User.class);
	        //条件,一个criteria中可以组合无数个条件，但是这些条件都是and关系
	        Example.Criteria criteria = example.createCriteria();
	        criteria.andEqualTo("sex","1");
	        criteria.andBetween("age",20,30);
	        List<User> users = userMapper.selectByExample(example);
	        System.out.println("users = " + users);
	
	    }



3.2 添加or条件的组合查询

	@Test
	    public void testSelectByExampleOr(){
	        //条件的组合工具
	        Example example = new Example(User.class);
	        //条件,一个criteria中可以组合无数个条件，但是这些条件都是and关系
	        Example.Criteria criteria = example.createCriteria();
	        criteria.andEqualTo("sex","1");
	        Example.Criteria criteria1 = example.createCriteria();
	        criteria1.andLike("name","%李%");
	        //直接使用example的or方法使得第二个条件和原来的条件组合or关系
	        example.or(criteria1);
	        System.out.println(userMapper.selectByExample(example));
	
	    }



3.3 排序

	@Test
	    public void testSelectByExampleOrder(){
	        //条件的组合工具
	        Example example = new Example(User.class);
	        //条件,一个criteria中可以组合无数个条件，但是这些条件都是and关系
	        Example.Criteria criteria = example.createCriteria();
	        criteria.andEqualTo("sex","1");
	        Example.Criteria criteria1 = example.createCriteria();
	        criteria1.andLike("name","%李%");
	        //直接使用example的or方法使得第二个条件和原来的条件组合or关系
	        example.or(criteria1);
	        //排序
	        example.setOrderByClause("id DESC");
	        System.out.println(userMapper.selectByExample(example));
	
	    }
