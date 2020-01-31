1. 解决创建maven项目过慢的问题
archetypeCatalog
internal

2. 创建出来的目录结构不全,需要对其进行补全

	在main目录下创建java和resources文件夹
	然后将其标记为对应类型的文件夹
3. 导入坐标
4. 设置一个前端控制器器,其实就是一个servlet,需要在web.xml中对其进行配置

	- 其实就是用于拦截请求
	- SpringMVC提供的DispatcherServlet类
	- /表示所有的请求都会被拦截到

5. 编写一个SpringMVC的配置文件