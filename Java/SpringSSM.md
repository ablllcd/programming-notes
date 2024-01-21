## SSM配置整合

整体思路为：

1. 导入所有SSM本身需要的依赖
2. 利用IOC容器来协作，由于Spring和SPring MVC本来就依赖于IOC容器，所以核心配置就是将Mybatis导入到Spring家族中
3. 每个SSM部分单独构建Config类，然后利用MVC中的那个启动配置类来将所有配置类加进去就行