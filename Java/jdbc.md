# Java DataBase Connectivety
## 概念
![Alt text](pic/jdbc.png)

JDBC是Java操作实际数据库的`接口`，其中只是定义了规范而没有真正实现如何操作数据库。实现是由各个数据库厂商负责的，他们会提供对应的`驱动jar包`。例如：

    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
	</dependency>

这样的好处就是可以封装底层的数据库操作，程序员直接面向jdbc接口写代码。

## 范例
````
	public void testJDBC() throws ClassNotFoundException, SQLException{
		// 1. 注册驱动
		Class.forName("com.mysql.cj.jdbc.Driver");

		// 2. 连接到数据库
		String url = "jdbc:mysql://localhost:3306/db01";
		String username = "root";
		String password = "123456";
		Connection connection = DriverManager.getConnection(url,username,password);
		
		// 3. 执行sql语句
		String sql = "select * from user";
		Statement statement = connection.createStatement();
		ResultSet resultSet = statement.executeQuery(sql);

		// 4. 封装结果数据
		List<User> list = new ArrayList<>();
		while (resultSet.next()) {
			int id = resultSet.getInt("id");
			String name = resultSet.getString("name");
			String gender = resultSet.getString("gender");

			User user = new User(id,name,gender);
			list.add(user);
		}

		// 5.释放资源
		statement.close();
		resultSet.close();

		list.forEach(e->{
			System.out.println(e);
		});
	}
````