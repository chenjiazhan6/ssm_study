# SSM整合步骤

###1. 配置好SpringMVC,能够处理请求并返回对应的页面


###2. 配置好Spring,配置好对应的配置文件


###3. 整合Spring和SpringMVC

1. 配置好SpringMVC启动服务器之后,并不会加载Spring,此时controller层需要的service层的对象并不会被创建和注入,所以此时需要在服务器启动
2. 配置了监听器,在tomcat中每一个项目都会创建一个ServletContext对象,监听器用于监听这个对象的创建,一旦创建,就扫描里面声明的对应的文件

		 <listener>
	        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	    </listener>
	    <context-param>
	        <param-name>contextConfigLocation</param-name>
	        <param-value>classpath:applicationContext.xml</param-value>
	    </context-param>

具体配置如上,因此,在项目部署时,Spring对应的配置文件便能被加载,对象也能被加到容器中去,这样便能成功整合SpringMVC和Spring

###4. 整合Spring和Mybatis
1. 需要让Spring生成接口的代理对象,创建一个数据源,创建一个SqlSessionFactoryBean用于创建一个SQLSessionFactory,同时声明配置文件所在位置
2. 扫描dao层中的接口,生成代理对象
3. SqlSessionFactory能够知道配置文件的位置,此时扫描dao层接口,自然能够生成对应的对象
	
			 <bean id="datasource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	        <property name="user" value="root"></property>
	        <property name="password" value="1234567890"></property>
	        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/ssm?serverTimezone=UTC"></property>
	        <property name="driverClass" value="com.mysql.cj.jdbc.Driver"></property>
	    </bean>
	
	<!--    创建出一个SqlSessionFactoryBean,并且能够声明xml对应的位置  -->
	    <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
	        <property name="dataSource" ref="datasource"></property>
	        <property name="mapperLocations" value="classpath:mapper/*.xml"></property>
	    </bean>
	
	<!--    扫描这个dao包,就能生成对应的动态代理的对象-->
	    <bean id = "mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	        <property name="basePackage" value="com.itheima.dao"></property>
	    </bean>	


## 需要注意的事项

### 父子容器的关系

 1. Controller层的注解有SpringMVC里进行扫描,将对象存储到SpringMVC的容器中去,而Service和dao层的对象就存储到Spring的容器中去,所以在Spring和SpringMVC中开启注解扫描需要使用子标签,配置如下

		- Spring的配置
	
		<context:component-scan base-package="com.itheima" >
	    <!-- 不扫描注解Controller -->
	        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	    </context:component-scan>

		- springMVC的配置
		
		<context:component-scan base-package="com.itheima" use-default-filters="false">
		      <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>  
		 </context:component-scan>

	 **在SpringMVC配置中需要将use-default-filters的值设置为false,不使用默认的过滤器,不然SpringMVC同样会扫描Service的注解,并将其加到容器中去,会覆盖掉Spring中的Service对象,Spring为Service层加上的事务就起不了作用,因为根本不会用到Spring容器中的Service对象**


2. SpringMVC和Spring两个容器之间的关系是父子关系,SpringMVC是子容器,Spring是父容器,子容器能够使用父容器中的对象,但是父容器不能使用子容器的对象

### 异常类型
1. java.lang.AbstractMethodError: Method com/mchange/v2/c3p0/impl/NewProxyResultSet.isClosed()Z is abstract
		

	这个是c3p0版本过低导致的,提高版本即可

2. org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.itheima.dao.UserDao.sava

	这个是绑定异常,很可能是没有xml文件与对应的接口相对应,或者是对应的xml文件与接口方法的名称不一致而引起的,特别是一个方法可以执行,另一个方法不能执行,肯定是xml文件中sql语句中id的值与接口方法名不一致而引起的

3. Artifact ssm:war: Error during artifact deployment. See server log for details.

	部署war包时直接出错,此时需要检查所有的配置文件
	比较好的方法是先配置好SpringMVC,然后启动,查看是否出错,如果不出错,则查看下一个配置Spring的配置
	肯定是某个位置写错
	
	今天错误类型是在Spring配置中配置SQLSessionFactoryBean时,需要使用ref来引用其他datasource对象但是却用value所导致的错误
		
	 <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" value="datasource"></property>
        <property name="mapperLocations" value="classpath:mapper/*.xml"></property>
    </bean>


