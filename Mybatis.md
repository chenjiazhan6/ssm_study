# Mybatis框架学习

## 对应项目:day01_mybatis_simple

## 1.入门程序

1. 编写两个配置文件,一个是创建用于SqlSessionFactory的文件,一个是用于动态生成接口代理类的mapper文件
2. 用于生成SqlSessionFactory的配置文件的配置,这里配置了数据库连接池等的配置

		<?xml version="1.0" encoding="UTF-8" ?>
		<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
		        "http://mybatis.org/dtd/mybatis-3-config.dtd">
		
		<configuration>
		    <!-- 和spring整合后 environments配置将废除 -->
		    <settings>
		        <setting name="lazyLoadingEnabled" value="true"/>
		        <setting name="aggressiveLazyLoading" value="false"/>
		        <!-- 控制台打印sql语句 -->
		        <setting name="logImpl" value="STDOUT_LOGGING"/>
		    </settings>
		    <environments default="development">
		        <environment id="development">
		
		            <!-- 使用jdbc事务管理 -->
		            <transactionManager type="JDBC" />
		            <!-- 数据库连接池 -->
		            <dataSource type="POOLED">
		                <property name="driver" value="com.mysql.cj.jdbc.Driver" />
		                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC" />
		                <property name="username" value="root" />
		                <property name="password" value="1234567890" />
		            </dataSource>
		        </environment>
		    </environments>
		<mappers>
 			 <mapper resource="dynamicMapper.xml"></mapper>
    	</mappers>


3. 用于生成代理对象的mapper文件,其中mapper标签中的属性namespace为接口的全类名,其中select或者update标签中的id为接口中对应的方法名

		<?xml version="1.0" encoding="UTF-8" ?>
		<!DOCTYPE mapper
		        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
		<!-- namespace：命名空间，类似于java包，主要用于隔离sql语句的，后续有重要作用
		     #{}：占位符，相当于jdbc的?
		     ${}:字符串拼接指令，注意如果入参为普通数据类型时，括号里面只能写value
		 -->
		<mapper namespace="com.itheima.domain.StudentMapper">
		    <!-- id:sql id标识sql语句的唯一标识
		             parameterType:入参的数据类型
		             resultType:返回结果的数据类型
		    -->
		    <select id="getStudentById" parameterType="int" resultType="com.itheima.domain.Student">
		        SELECT * FROM student where id = #{id};
		    </select>
		</mapper>


4. 代码编写

		
		public static SqlSessionFactory getSqlSessionFactory() throws IOException {
	        InputStream ins = Resources.getResourceAsStream("sqlFactory.xml");
	        SqlSessionFactory sqlSessionFactory =  SqlSessionManager.newInstance(ins);
	        return sqlSessionFactory;
	    }
	 	public void test02() throws IOException {
	       // System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
	        SqlSession sqlSession = getSqlSessionFactory().openSession();
	        StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
	        Student stu = studentMapper.getStudentById(1);
	        System.out.println(stu);
	    }
5. 创建SqlSession需要注意的内容

		需要注意的是 SqlSession sqlSession = getSqlSessionFactory().openSession()这种方式创建出    
		来的SqlSession是需要手动提交的,不会自动提交的,需要手动进行提交
		SqlSession sqlSession = getSqlSessionFactory().openSession(true)这种方式创建出来的SqlSession将会自动提交


##2. 获取参数

### 1. 总体说明

	mybatis同样是通过动态代理来生成代理对象,但是在invocationHandler中的invoke方法中,如果是接口外的方法直接放行
    但是如果是接口中的方法,将会对传入的参数进行封装,封装为一个map
    在使用代理对象来执行对应的增删改操作时,传入的参数会在invoke方法处被封装,封装为一个map,再到动态
    生成的方法内部执行,执行方法内部的sql语句

### 2. 获取自增主键

-  方法说明

			在mybatis中可以设置属性useGeneratedKeys为true,同时通过keyProperty指定自动生成的主键值   
			要赋给传入的对象中哪一个属性,这样传入的对象中的属性便能赋值为主键的值,在方法获取参数前Mybatis会先对参数进行封装

- 代码示例
	
	- 接口类
	
				public interface UserDao {
				    void insert(Student stu);
				}
	
	- mapper文件中配置的编写
	
				<mapper namespace="pratices.GeneratedKey.UserDao">
				   <insert id="insert" parameterType="com.itheima.domain.Student" useGeneratedKeys="true" keyProperty="id">
				       INSERT INTO student VALUES(id,name,gender,year)
				   </insert>
				</mapper>

	- 测试代码

				public void test05() throws IOException {
			        SqlSession sqlSession = getSqlSessionFactory().openSession(true);
			        UserDao userDao = sqlSession.getMapper(UserDao.class);
			        Student stu = new Student(null, "fff", "0", 32);
			        userDao.insert(stu);
			        System.out.println(stu.toString());
			    }

- 结果:上面测试代码中对象stu中的id便被赋值为自增主键的值


###3. 参数获取方式


1. 单个参数的获取
		
	- 获取方法

			mybatis不会做特殊处理
			#{参数名}:取出参数值  
			这个参数名可以随意编写,参数都可以成功获取


	- 接口方法

			Student findOne(int id);//单个参数
		
			
	- mapper文件配置的编写,从下面可看出获取单个参数时名称可随意编写

				<mapper namespace="Param.TestParam">
				    <select id="findOne" parameterType="int" resultType="com.itheima.domain.Student">
				        SELECT * FROM student where id = #{fqfqefq}
				    </select>
				</mapper>

	- 测试代码

			public void test05() throws IOException {
		        SqlSession sqlSession = getSqlSessionFactory().openSession(true);
		        TestParam testParam = sqlSession.getMapper(TestParam.class);
		        Student stu = testParam.findOne(2);
		        System.out.println(stu);
		    }


2. 多个参数

	- 获取方法

			多个参数会被封装成一个map
		    key:param1...paramN,或者参数的索引（0,1...)也行
		    value:传入的参数值
		    #{}就是从map中获取指定key的值
		    可以通过在dao层的接口方法明确指定mybatis封装多个参数为map时key的取值,可以使用@Param("")注解

	
	- 接口方法

			Student findStuByIdAndName(@Param("id") int id, @Param("name")String name);
		
	- mapper配置文件的编写
	
		- 没有进行任何设置的话,mybatis默认参数在map中的key为param1...或者0...
	
	
			    <select id="findStuByIdAndName" resultType="com.itheima.domain.Student">
			      SELECT * FROM student WHERE id = #{param1} AND name = #{param2}
			    </select>

		- 在接口方法中加入@Param注解的话,Mapper文件获取参数可以使用@Param注解处指明的参数名的值

				Student findStuByIdAndName(@Param("id") int id, @Param("name")String name);
			   
			    <select id="findStuByIdAndName" resultType="com.itheima.domain.Student">
			      SELECT * FROM student WHERE id = #{id} AND name = #{name}
			    </select>

	- 测试代码的编写

			public void test() throws IOException {
		        SqlSession sqlSession = getSqlSessionFactory().openSession(true);
		        TestParam testParam = sqlSession.getMapper(TestParam.class);
		        Student stu = testParam.findStuByIdAndName(1,"ddd");
		        System.out.println(stu.toString());
		    }

3. 参数为POJO对象:方法中存在多个参数,如果多个参数刚好是POJO对象的属性,那么直接传入一个POJO即可

	- 获取参数的方法

			直接使用#{POJO中的属性名即可}
			对应的接口方法:
			Student findStuByPojo(com.itheima.domain.Param param);

	- 接口方法

			Student findStuByPojo(com.itheima.domain.Param param);

	- POJO类

			public class Param {
	
			    private Integer id;
			    private String name;
			
			    public Param(Integer id, String name) {
			        this.id = id;
			        this.name = name;
			    }
			
			    public Integer getId() {
			        return id;
			    }
			
			    public void setId(Integer id) {
			        this.id = id;
			    }
			
			    public String getName() {
			        return name;
			    }
			
			    public void setName(String name) {
			        this.name = name;
			    }
			}

	
	- mapper文件的编写


			<select id="findStuByPojo" resultType="com.itheima.domain.Student" parameterType="com.itheima.domain.Param">
			     SELECT * FROM student WHERE id = #{id} AND name = #{name}
		    </servlet>

	- 测试代码

			 public void test06() throws IOException {
		        SqlSession sqlSession = getSqlSessionFactory().openSession(true);
		        TestParam testParam = sqlSession.getMapper(TestParam.class);
		        Param param = new Param(1,"ddd");
		        Student stu = testParam.findStuByPojo(param);
		        System.out.println(stu.toString());
		    }



4.  如果不是POJO,那么直接传入一个map对象	

	- 参数获取的方法

			在map中key为多少,在配置文件中便使用相对应的key来获取对应的属性

	- 接口方法

			Student findStuByMap(Map<String,Object> map);
	
	- mapper配置文件的编写

			<select id="findStuByMap"  parameterType="map" resultType="com.itheima.domain.Student">
		       SELECT * FROM student WHERE id = #{id} AND name = #{name}
		    </select>

	- 测试代码

			 public void test07() throws IOException {
		        SqlSession sqlSession = getSqlSessionFactory().openSession(true);
		        TestParam testParam = sqlSession.getMapper(TestParam.class);
		        Map<String,Object> map = new HashMap<String,Object>();
		        map.put("id",1);
		        map.put("name","ddd");
		        Student stu = testParam.findStuByMap(map);
		        System.out.println(stu);
		    }

5. 参数为Collection类型

	- 参数获取方法

			如果参数是Collection(List,set)类型或者是数组类型,也会对参数进行特殊处理,即使只有一个参数,mybatis也会对其进行参数处理
			Collection类型使用的key为collection,如果是List也可以使用这个key:list
		    数组key:array
	
	- 接口方法

			List<Student> findStuByList(List<Integer> idList);

	- mapper配置文件的编写

			 <select id="findStuByList" parameterType="list" resultType="com.itheima.domain.Student">
		        SELECT * FROM student WHERE id = #{list[0]} OR id = #{list[1]}
		    </select>

	- 测试代码的编写

			public void test08() throws IOException {
		        SqlSession sqlSession = getSqlSessionFactory().openSession(true);
		        TestParam testParam = sqlSession.getMapper(TestParam.class);
		        List<Integer> list = new ArrayList<Integer>();
		        list.add(1);
		        list.add(2);
		        List<Student> studentList = testParam.findStuByList(list);
		        for(Student stu : studentList){
		            System.out.println(stu.toString());
		        }
		    }


6. 参数值获取方式
	    
	- #{}:以预编译的方式通过指明key获取map集合中属性的值,相当于使用PrepareStatement,防止sql注入
	- ${}:直接从map集合中按照key的值取出map中对应的value注入sql语句中,存在安全问题
	- 大多数情况下都使用#{}这种方式来获取数据,当原生JDBC不支持占位符的形式,那么就可以使用${}来将数据取出


## 4. 数据的封装

### resultType的使用

1. 使用Mybatis提供的自定义的数据的封装

	- 接口方法

		 	Student SimpleDemo(Integer id);

	- mapper文件的编写,直接封装为Student类型

			 <select id="SimpleDemo" resultType="com.itheima.domain.Student">
		        SELECT * FROM student WHERE id = #{id}
		    </select>

	- 测试代码

			public void test() throws IOException {
		        SqlSession sqlSession = getSqlSessionFactory().openSession();
		        ReturnValueTest  returnValueTest = sqlSession.getMapper(ReturnValueTest.class);
		        Student stu = returnValueTest.SimpleDemo(1);
		        System.out.println(stu);
		    }
		
	- 具体说明

			mybatis对数据进行封装其实是通过反射来对数据进行封装的,如果我们通过使用默认的方式对类进行  
			数据的封装,那么此时将会根据数据库中的字段名,通过反射来调用setter方法
			如果数据库返回数据的字段与返回类中字段名相同,那么就能够成功封装,如果不一致的话,那就不能成功封装

2. 将记录使用map形式展示出来,key为数据库的列名,value为其对应的值
    
	- 接口方法
	
			 Map<String,Object> oneToMap(int id);

	- mapper文件的编写,返回的数据类型为map
	
			 <select id="oneToMap" resultType="map">
		       SELECT * FROM student WHERE id = #{id}
		  	 </select>
	- 测试代码,数据将以键值对的方式直接返回


			 public void test04() throws IOException {
		        SqlSession sqlSession = getSqlSessionFactory().openSession();
		        ReturnValueTest  returnValueTest = sqlSession.getMapper(ReturnValueTest.class);
		       Map<String,Object> map = returnValueTest.oneToMap(1);
		        System.out.println(map.toString());
		    }


3. 当返回多条记录时,要将多条记录直接封装为一个map,要求使用数据库中主键的值作为key(自身想要使用数据库哪一列均可),封装的javaBean作为value  
此时返回的returnType直接编写为返回的javaBean类型,但是还要在接口方法中使用注解@Mapkey()告知mybatis使用数据库中哪一个属性作为key

	- 接口方法,将多条记录封装成一个map,在接口方法处使用@MapKey注解指定map中的key

			  @MapKey("id")//使用数据库中数据的值
	    	  Map<Object,Object> listToMap(List<Integer> list);

	- mapper文件的编写

			<select id="listToMap" resultType="com.itheima.domain.Student" parameterType="list">
		        SELECT * FROM student WHERE id = #{list[0]} OR id = #{list[1]}
		    </select>

	- 测试代码

			public void test05() throws IOException {
		        SqlSession sqlSession = getSqlSessionFactory().openSession();
		        ReturnValueTest  returnValueTest = sqlSession.getMapper(ReturnValueTest.class);
		        List<Integer> list = new ArrayList<Integer>();
		        list.add(2);
		        list.add(1);
		        Map<Object,Object> returnMap = returnValueTest.listToMap(list);
		        System.out.println(returnMap);
		    }

### ResultMap的使用

####1. 概述:可以使用resultMap来自定义封装的规则

#### 基本使用

1. 使用方法

		
	- 外部标签:
		- id属性唯一标识这个自定义封装规则
		- type属性声明查询结果将被封装为什么类型
		
	- 内部标签:
		- id子标签:定义主键列的封装规则
		- result子标签:用于声明普通列的封装规则
			- column:声明数据库中对应的列
			- property:声明封装到哪一项属性中
	- 附加说明
	
			<id>定义主键在底层会有优化
			在resultMap定义封装规则时,如果不指定的话,那么查询结果将会自动进行封装
			但是如果我们编写resultMap,那么应该将规则编写完整

2. 接口方法

	 Student findById(int id);

3. mapper文件编写示例

		  <select id="findById" resultMap="studentResultMap">
	        SELECT * FROM student WHERE id = #{id}
	      </select>
	      <resultMap id="studentResultMap" type="com.itheima.domain.Student">
	          <id  column="id" property="id"></id>
	          <result column="name" property="lastName"></result>
	          <result column="gender" property="gender"></result>
	          <result column="year" property="year"></result>
	      </resultMap>

4. 测试方法

		public void test01() throws Exception {

	        SqlSession sqlSession = getSqlSessionFactory().openSession();
	        UseResultMap useResultMap =  sqlSession.getMapper(UseResultMap.class);
	        Student stu = useResultMap.findById(1);
	        System.out.println(stu.toString());
	
	    }


#### 复合属性的封装
1. 要求

 - 实际要求
	
	       - 查询员工时,同时查询出员工所在的部门信息,员工中存在一个部门属性,查询出来的某一列是部门属性的子属性可以直接使用级联属性来进行数据的封装 
	      
			
 - 待封装的类

			public class StudentPlus {
			    private Integer id;
			    private String lastName;
			    private String gender;
			    private int year;
			    private Department dept;
			}

2. 实现方法

	可以使用关联标签对属性进行封装,这个association标签是用于封装关联对象的子标签

		 
		<association property="" javaType=""></association>
		- property:待封装的属性的名字
		- javaType:对象的具体类型

3. mapper文件编写范例

		<select id="findByIdPlus" resultMap="studentPlusResultMap">
	        select stu.id stu_id,stu.name stu_name,stu.gender stu_gender,stu.year stu_year,
	        dept.id dept_id,dept.name dept_name FROM student stu inner join dept on  stu.dept_id = dept.id where stu.id = #{id};
	    </select>
	
	    <resultMap id="studentPlusResultMap" type="com.itheima.domain.StudentPlus">
	        <id  column="stu.id" property="id"></id>
	        <result column="stu.name" property="lastName"></result>
	        <result column="stu.gender" property="gender"></result>
	        <result column="stu.year" property="year"></result>
	        <association property="dept" javaType="com.itheima.domain.Department">
	            <id column="dept_id" property="id"></id>
	            <result column="dept_name" property="name"></result>
	        </association>
	    </resultMap>

	
4. 分步查询

	- 分析

		    可以先在员工表中找出dept的id,再使用该id找出dept的值来找出公寓的值

	- mybatis提供的标签:

			<association property="" select="" column=""></association>

			- property:select查询封装出来的结果存储到property属性中
			- select:调用某一个select
			- column:指明要传入的值,格式为{key1=数据库某列的值,key2=数据库某列的值}
 
 	     	使用这个标签可以成功将查询的结果封装后存储到某个属性中去

	- mapper文件编写

			 <select id="findStuPlusByIdStep" resultMap="studentPlusStep">
		        SELECT * FROM student where id = #{id}
		    </select>
		
		    <resultMap id="studentPlusStep" type="com.itheima.domain.StudentPlus">
		        <id  column="stu.id" property="id"></id>
		        <result column="stu.name" property="lastName"></result>
		        <result column="stu.gender" property="gender"></result>
		        <result column="stu.year" property="year"></result>
		        <association property="dept" select="deptById" column="{id=dept_id}"></association>
		    </resultMap>
		
		    <select id="deptById" resultType="com.itheima.domain.Department">
		        SELECT * FROM dept WHERE id = #{id}
		    </select>

	- 测试代码

			public void test03() throws Exception {

		        SqlSession sqlSession = getSqlSessionFactory().openSession();
		        UseResultMap useResultMap =  sqlSession.getMapper(UseResultMap.class);
		        StudentPlus stu = useResultMap.findStuPlusByIdStep(100);
		        System.out.println(stu.toString());
		
		    }


#### 列表属性封装

1. 要求
	
	- 要求查出部门中所有的员工,使用多表联查,当要封装的属性为集合属性时

2. 待封装的类


		public class Department {
	
		    private int id;
		    private String name;
		    private List<Student> list;
		}

3. mybatis提供的标签:collection


		可以使用collection标签来对集合属性进行封装
	    <collection property="" ofType=""></collection>
		- property:待封装类的属性
		- ofType:集合中的元素类型

4. mapper文件的编写

		 <select id="findDeptPlus" resultMap="DeptResultMap">
	             select stu.id stu_id,stu.name stu_name,stu.gender stu_gender,stu.year stu_year,
	        dept.id dept_id,dept.name dept_name FROM student stu inner join dept on  stu.dept_id = dept.id where dept_id = #{dept_id}
	    </select>
	
		上面类属性的封装
	    <resultMap id="DeptResultMap" type="com.itheima.domain.Department">
	        <id column="dept_id" property="id"></id>
	        <result column="dept_name" property="name"></result>
	        <collection property="list" ofType="com.itheima.domain.Student">
	            <id  column="stu_id" property="id"></id>
	            <result column="stu_name" property="lastName"></result>
	            <result column="stu_gender" property="gender"></result>
	            <result column="stu_year" property="year"></result>
	        </collection>
	    </resultMap>

5. 测试代码

		 public void test04() throws Exception {
	
	        SqlSession sqlSession = getSqlSessionFactory().openSession();
	        UseResultMap useResultMap =  sqlSession.getMapper(UseResultMap.class);
	        Department stu = useResultMap.findDeptPlus(1);
	        System.out.println(stu.toString());
	        //  System.out.println(stu.toString());
	
	    }


6.分步查询实现功能

 - 标签说明
	
	    <collection property="" select="" column=""></collection>

  	 	- 在这个collection标签内部使用select属性使用另一个查询
   	    - column是要求传入的值的数据库对应的列
		- property: 要封装到哪个属性	

- mapper文件的编写

		<select id="findDeptPlusStep" resultMap="DeptResultMapStep">
	        SELECT * FROM dept WHERE id = #{id}
	    </select>
	    <select id="findStudentByDeptId" resultType="com.itheima.domain.Student">
	        SELECT * FROM student WHERE dept_id = #{id}
	    </select>
	    <resultMap id="DeptResultMapStep" type="com.itheima.domain.Department">
	        <id column="id" property="id"></id>
	        <result column="name" property="name"></result>
	        <collection property="list" select="findStudentByDeptId" column="id" >
	        </collection>
	    </resultMap>



## 动态sql的使用

### if标签

1. 需求说明

		查询员工,传入整个对象,哪个字段存在就根据这个字段进行查询

2. 使用标签

		 if标签,在test属性内部编写OGNL表达式,这个表达式成立才会将该字符串进行拼凑,否则不拼凑
		 编写表达式时遇到特殊字符,则需要使用转译字符
		  <if test="">
		
		  </if>

3. mapper文件编写

		<select id="testWhere" resultMap="studentResultMap">
	          SELECT * FROM student WHERE
	           <if test="id != null">id = #{id}</if>
	           <if test="lastName != null">AND name = #{lastName}</if>
	           <if test="gender != null">AND gender = #{gender}</if>
	           <if test="year != null">AND year = #{year}</if>
	         
	     </select>

4 测试代码
	

		public void testIf() throws IOException {
	        SqlSession sqlSession = getSqlSessionFactory().openSession();
	        DynamicUse studentMapper = sqlSession.getMapper(DynamicUse.class);
	        Student student = new Student(null,"ddd",null,22);
	        Student stu = studentMapper.testif(student);
	        System.out.println(stu);
	    }
		

### where标签
1. 解决问题

		但是如果第一个if中的表达式就不成立,此时将会报错

2. 解决方法
		此时存在两种解决方案:
		1. 在where后面先加上一个恒成立的条件
		2. where不再手动编写,而是使用where标签,此时where标签会去掉第一个多余的and或者or

3. mapper文件编写

		<select id="testWhere" resultMap="studentResultMap">
	          SELECT * FROM student
	          <where>
	           <if test="id != null">id = #{id}</if>
	           <if test="lastName != null">AND name = #{lastName}</if>
	           <if test="gender != null">AND gender = #{gender}</if>
	           <if test="year != null">AND year = #{year}</if>
	          </where>
	      </select>


3. 测试代码

		public void testWhere() throws IOException {
	        SqlSession sqlSession = getSqlSessionFactory().openSession();
	        DynamicUse studentMapper = sqlSession.getMapper(DynamicUse.class);
	        Student student = new Student(null,"ddd",null,22);
	        List<Student> stu = studentMapper.testWhere(student);
	        System.out.println(stu);
	    }

### trim标签

1. 问题

 		无法去掉最后一个多余的and或者or

          

2. 解决方法

		- 使用where标签编写sql要规范
		- 使用trim标签用于解决这个问题

3. trim标签说明

		<trim prefix="" prefixOverrides="" suffix="" suffixOverrides=""></trim>
	
		 prefix:在标签内部生成的语句整体加上一个前缀
		 prefixOverrides:将生成语句的多余的字符去掉
		 suffix:在生成的语句整体加上一个后缀
		 suffixOverrides:将生成的语句后面多余的字符串去掉
		
		 这样就能够解决 后面/前面 多余的 and/or 的内容

4. mapper文件编写

		<select id="testTrim" resultMap="studentResultMap">
	        SELECT * FROM student
	        <trim prefix="WHERE" suffixOverrides="AND">
	            <if test="id != null">id = #{id} AND</if>
	            <if test="lastName != null"> name = #{lastName} AND </if>
	            <if test="gender != null">gender = #{gender} AND</if>
	            <if test="year != null"> year = #{year}</if>
	        </trim>
	    </select>

5. 测试代码

		public void testTrim() throws IOException {
	        SqlSession sqlSession = getSqlSessionFactory().openSession();
	        DynamicUse studentMapper = sqlSession.getMapper(DynamicUse.class);
	        Student student = new Student(null,"ddd",null,null);
	        List<Student> stu = studentMapper.testTrim(student);
	        System.out.println(stu);
	    }


### choose标签的使用
1. 问题

	只使用其中一个查询条件,当满足test内部编写的OGNL表达式,才会使用这个条件
   
2. choose标签说明:choose标签的使用相当于switch-case的使用
		 
		  <choose>
		         <when test="">
		
		         </when>
		          <otherwise>
		
		          </otherwise>
		   </choose>
		   只使用其中一个查询条件,当满足test内部编写的OGNL表达式,才会使用这个条件
		   就是查询时只使用其中一个条件
		   而<otherwise>标签就相当于switch中的default,如果上面所有的条件都不满足,那就使用otherwise标签内部所使用的条件

3. mapper文件编写


		 <select id="testChoose" resultMap="studentResultMap">
	        SELECT * FROM student where
	        <choose>
	            <when test="id != null">id = #{id}</when>
	            <when test="lastName != null"> name = #{lastName} </when>
	            <when test="gender != null">gender = #{gender} </when>
	            <when test="year != null"> year = #{year}</when>
	            <otherwise>1 = 1</otherwise>
	      </choose>
	    </select>

4. 测试代码

		public void testChoose() throws IOException {
	        SqlSession sqlSession = getSqlSessionFactory().openSession();
	        DynamicUse studentMapper = sqlSession.getMapper(DynamicUse.class);
	        Student student = new Student(null,"ddd",null,null);
	        List<Student> stu = studentMapper.testChoose(student);
	        System.out.println(stu);
	    }

### foreach标签

1. 问题

		遍历传入的list集合

2. foreach标签说明

		<foreach collection="" item="item_id" separator="" open="" close="">
	            #{item_id}
	    </foreach>
	    collection:指定要遍历的集合
	    list类型的参数会特殊处理封装在map,map的key就叫做list
	    item:将遍历出的元素赋给指定的变量
	    separator:指明遍历出来元素之间的分割符
	    open:结果集拼接上一个开始符号
	    close:遍历出来的结果集拼接上一个结束符号

3. mapper文件编写

		 <select id="testForeach" resultMap="studentResultMap" parameterType="list">
	        SELECT * FROM student where id in
	        <foreach collection="list" open="(" close=")" separator="," item="data" >
	            #{data}
	        </foreach>
	    </select>

4. 测试代码

		public void testForeach() throws IOException {
	        SqlSession sqlSession = getSqlSessionFactory().openSession();
	        DynamicUse studentMapper = sqlSession.getMapper(DynamicUse.class);
	       // Student student = new Student(null,"ddd",null,null);
	        List<Integer> list = new ArrayList<Integer>();
	        list.add(1);
	        list.add(2);
	        List<Student> stu = studentMapper.testForeach(list);
	        System.out.println(stu);
	    }