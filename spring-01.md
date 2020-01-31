# Spring框架学习

## 第一章:创建对象与依赖注入

### 1. Spring中基于XML的IOC环境搭建

	1. 创建一个Spring的配置文件,名为bean.xml
	2. 在配置文件中引入最基本的约束


			<?xml version="1.0" encoding="UTF-8"?>
			<beans xmlns="http://www.springframework.org/schema/beans"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://www.springframework.org/schema/beans
	         http://www.springframework.org/schema/beans/spring-beans.xsd">

			    <bean id = "ServiceDaoImpl" class="com.dao.ServiceDaoImpl"></bean>
			
			    <bean id="ServiceImpl" class="com.service.ServiceImpl"></bean>

			</beans>

	3.  在配置文件中使用bean标签来管理对象  
	4. 代码使用Spring容器示例

			  ApplicationContext applicationContext =  new ClassPathXmlApplicationContext("bean.xml");
	          IServiceDao iServiceDao = (IServiceDao) applicationContext.getBean("ServiceDaoImpl");
	          IService service = (IService)applicationContext.getBean("ServiceImpl") ;
	          System.out.println(iServiceDao);
	          System.out.println(service);


### 2. 在spring配置文件中创建对象的三种方式

		
1. 使用默认构造函数创建对象
			
	- 待创建的java类
			
			public class ServiceImpl implements IService {
			    public void save() {
			        System.out.println("方法被执行了");
			    }
			}		

	- 在spring配置文件中的内容,使用默认构造函数创建出ServiceImpl对象
	
		  	<bean id="ServiceImpl" class="service.ServiceImpl" scope="singleton"></bean>

2. 使用某个类中的方法创建对象

	- 创建对象的类

		public class methodFactory {
		
		    public IService getService(){
		        return new ServiceImpl();
		    }
 			}


	- 在配置文件中,使用getService方法创建出ServiceImpl对象

			- 先创建出methodFactory这个对象
			<bean id = "methodFactory" class="Factory.methodFactory"></bean>
			- 再使用methodFctory对象中的getServic方法创建出ServiceImpl对象
	    	<bean id = "methodbean" factory-bean="methodFactory" factory-method="getServic"></bean>

3.  使用某个类的静态方法创建对象

	 - 静态工厂类
	 
			public class staticFactory {
	
			    public static IService getServic(){
			        return new ServiceImpl();
			    }
			}
	  
	 - 在配置文件中使用该静态方法创建对象

			<bean id = "staticFactory" class="Factory.staticFactory" factory-method="getServic"></bean>

 
###  3. bean的作用范围和生命周期

1. 对象的作用范围:

	         1. singleton:单例的
	         2. prototype:多例的
	         3. request:web应用下的请求对象
	         4. session:web应用下的session范围
	         5. global-session:作用于集群环境下的session


2.  bean的生命周期:


	         1. 单例对象:
	            出生:容器创建时对象自动创建
	            活着：容器活着它就活着
	            销毁：容器销毁时便销毁
	            容器中的对象与容器同生共死
	
	         2. 多例对象
	            出生:当使用到对象时对象就会被创建
	            活着：使用过程中就活着
	            销毁：由垃圾回收器对其进行统一回收


###  4. 依赖注入:在对象属性中注入对应的值

1.  使用构造函数
	
	- java类

			public class ServiceImpl implements IService {
			    private int age;
			    private String name;
			    private Date birthday;
			
			    public ServiceImpl(int age, String name, Date birthday) {
			        this.age = age;
			        this.name = name;
			        this.birthday = birthday;
			    }
			}

	- 配置文件中的内容
	
			 <bean id = "ServiceImpl" class="com.service.ServiceImpl" >
	                    <constructor-arg name="name" value="李四"></constructor-arg>
	                    <constructor-arg name="age" value="20"></constructor-arg>
	                    <constructor-arg name="birthday" ref = "birthday"></constructor-arg>
	         </bean>
			 <bean id = "birthday" class="java.util.Date"></bean>
			name:指向对象中待注入数据的属性名
			value：普通类型和String类型
			ref:指向容器中其他的对象

2. 使用setter方法	

	- 对应的java类

		
			public class ServiceImpl implements IService {
				    private int age;
				    private String name;
				    private Date birthday;
			
					public int getAge() {
				        return age;
				    }
				
				    public void setAge(int age) {
				        this.age = age;
				    }
				
				    public String getName() {
				        return name;
				    }
				
				    public void setName(String name) {
				        this.name = name;
				    }
				
				    public Date getBirthday() {
				        return birthday;
				    }
				
				    public void setBirthday(Date birthday) {
				        this.birthday = birthday;
				    }
	
				}
			

	- 配置文件中的配置

			<bean id = "ServiceImpl2" class="com.service.ServiceImpl2" >
	                    <property name="age" value="22"></property>
	                    <property name="birthday" ref="birthday"></property>
	                    <property name="name" value="李四"></property>
	        </bean>
			<bean id = "birthday" class="java.util.Date"></bean>

	- 注意
	
		    1.底层使用反射,所以直接通过配置文件中属性的名字找到对应的setter方法,所以name属性的值与  
			  对象的属性名并没有必然的关系,而与对象中的setter方法相关
			2.举个例子来说,设置year属性的setter方法名为setYear,那么此时标签property属性name的值为   
			  year,但是假如setter方法名为setMyYear,那么此时property属性的name的值就变为myYear


## 2. Spring常用注解的使用

### 1. Spring启动注解扫描的环境搭建

1)具体说明

	- 加入新的约束
	- 使用标签<context:component-scan base-package="" />开启注解扫描

2)配置文件的配置

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xmlns:context="http://www.springframework.org/schema/context"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans
	                http://www.springframework.org/schema/beans/spring-beans.xsd
	                http://www.springframework.org/schema/context
	                http://www.springframework.org/schema/context/spring-context.xsd">
	
	    <context:annotation-config/>

	  <context:component-scan base-package="com.spring.annoation.service"></context:component-scan>
	</beans>


### 2. 创建对象的注解


		Component注解：默认创建的bean对象的名字为将类名首字母小写
	    下面三个注解与上面注解的功能相同,不同是用于三层架构中,使属于三层中的类能够更加清晰得辨识出来

        @Controller：表现层
        @Repository:持久层
        @Service:业务层

###3. 属性注入的注解


1. @Autowired和@Qualifer注解的使用

	- Autowired

	   - 具体说明
        
	       - 如果容器中中存在与要注入属性的变量具有相同类型的对象,那就使用容器中的对象自动注入到变量中
	       如果容器中不存在与将要注入变量具有相同类型的对象,那么将会报错    
	       - 这里有一个问题,如果存在多个同种类型但是id却不相同的bean,此时要求被注入的类变量的名称与bean的id
	       值相等才能成功注入,不然就失败了  
	       - 在使用注解时,此时set方法就不是必须的了,因为此时并非是使用反射来调用方法对变量进行赋值

		- 具体使用
	
				  @Autowired
			   	  private IServiceDao serviceDao = null;
			  

	- @Qualifier注解

		- 具体说明
        
		      - 可以使用@Qualifier(value="serviceDao")与 @Autowired注解配套使用,可以使用该注  
			     解指定类变量注入指定id的bean对象，@Qualifier注解单独使用将会报错,只有与  
			     @Autowired  配套使用才能生效
		        
		      - 当@Autowired注解与@Qualifer注解配合使用时,将先到容器中寻找符合类型要求的对象,然  
			     后将所筛选出的bean的id与@Qualifer注解中声明的值进行比较,相等就将该对象注入到变   
			     量中,如果找不到就失败

		- 具体代码

				  @Autowired
			      @Qualifier(value="serviceDao")
			      private IServiceDao serviceDao = null;


2. @Resource注解 
			
	- 具体说明
 		
		 - Resource注解注入指定id的bean对象

         - 因为@Resource(name = "serviceDao")可以直接根据id来指定bean对象,所以如果指定的类型与实际要注入的类型不一致的话,就会直接报错
        
         - 而对于  
            @Autowired  
            @Qualifier(value="serviceDao")  
            则会找到容器中要注入的类的对象,所以此处不会由于注入错误类型而导致错误的情况

	- 代码示例:

			@Resource(name = "serviceDao")
  		    private IServiceDao serviceDao = null;
        
3. 注意:上面的注解只能注入bean类型,不能注入普通类型或者是String类型而对于复杂类型,比如array,set等的属性需要xml配置才能注入,不能基于注解的注入


4. 普通类型及String类型属性的注入,可以使用@Value注解

		@Value(value = "22")
	    private int age;
	    @Value(value = "nihao")
	    private String name;



### 4. 作用范围的注解

- 具体说明
	
		 Scope注解：用于指定作用范围,常用属性为singleton,prototype,一个是单例一个是多例,不写默认是单例的

- 代码示例
	  
 		 @Scope(value = "singleton")
	     public class ServiceImpl implements IService {
	            。。。。代码的具体实现
	     }


### 5. 生命周期的注解

- 具体说明
		   
		   @PostConstruct对象创建时将会执行的方法
           @PreDestroy对象销毁时将会执行的方法
		   

- 代码示例

	在类中对应的方法加上@PostConstruct注解,在创建这个对象时,便会执行这个方法   
	加上@PreDestroy注解,销毁这个对象时,便会执行销毁方法

		public class ServiceImpl implements IService {

		   @PostConstruct
		   public void init() {
		       System.out.println("对象开始创建");
		   }
		   @PreDestroy
		   public void destroy() {
		        System.out.println("对象的销毁开始执行");
		   }

		}

### 6. Spring和JUnit的整合

1. 分析前的说明
	
		1. 应用程序的入口
	        main方法
	    2. junit没有main方法也可以成功运行
	        junit集成了一个main方法,这个方法会当前测试类中哪些方法存在@Test注解
	        便让有@Test注解的方法执行
	        使用invoke方法来调用方法
	    3. junit不会察觉到是否采用了Spring框架,所以也不会创建IOC核心容器
	    4. 所以即使是使用@Autowired注解也不会成功注入对应的对象
    
    
2. 解决方法
    
			使用新注解,将junit中的main方法创建出IOC核心容器,替换原先的main方法,使得main方法在执行方法前先创建出Spring容器
		    @RunWith(SpringJUnit.class)   
		    @ContextConfiguration(locations/classes),其中参数用于告知配置类或者是xml的位置
	        location:用于告知配置文件的位置,classpath表明是位于类路径下的
	        classes:用于告知配置类的具体类

3. 具体示例

	举例说明:  
    (1)Spring使用xml文件和注解结合开发的方式
    
        @RunWith(SpringJUnit4ClassRunner.class)
        @ContextConfiguration(locations = "classpath:bean.xml")
        public class Test {
            @Autowired
          private  IAccountService iAccountService;
            @org.junit.Test
            public void test01() throws SQLException {
                List<Account> list = iAccountService.findAll();
                for(Account account : list){
                    System.out.println(account.toString());
                }
           }
        }
    
    (2)使用纯注解的配置方式
    
        @RunWith(SpringJUnit4ClassRunner.class)
        @ContextConfiguration(classes = SpringConfig.class)
        public class Test {
        
            @Autowired
            private IAccountService accountService ;
            @org.junit.Test
            public void test01() throws SQLException {
        
                List<Account> list = accountService.findAll();
                for(Account account : list){
                    System.out.println(account.toString());
                }
            }
        }
          