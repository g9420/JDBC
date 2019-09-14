# JDBC 封装工具类(oracle 数据库)

## 版本1（简单连接数据库执行sql语句）
### 目录结构
#### ![](README_files/1.jpg)
### 在开始之前我们先回顾一下jdbc连接数据库的6个步骤
	1. 加载数据库驱动到jvm虚拟机，通过 Class类的静态方法.forName()
		* 例如
			 try{
					//加载oracle的驱动类    
					Class.forName("oracle.jdbc.OracleDriver") ;    
				}catch(ClassNotFoundException e){    
					System.out.println("找不到驱动程序类 ，加载驱动失败！");    
					e.printStackTrace() ;    
				}    
	2. 创建数据库的连接 
		* 要连接数据库，需要向java.sql.DriverManager请求并获得Connection对象，该对象就代表一个数据库的连接。  
		* 使用DriverManager的getConnectin(String url , String username ,  String password )方法传入指定的欲连接的数据库的路径、数据库的用户名和密码来获得

	3. 创建一个preparedStatement对象
		* 要执行SQL语句，必须获得java.sql.Statement实例，Statement实例分为以下3 种类型：    
			1. 执行静态SQL语句。通常通过Statement实例实现。    
			2. 执行动态SQL语句。通常通过PreparedStatement实例实现。    
			3. 执行数据库存储过程。通常通过CallableStatement实例实现。    
		* 具体的实现方式：    
			Statement stmt = con.createStatement() ;    
			PreparedStatement pstmt = con.prepareStatement(sql) ;    
			CallableStatement cstmt = con.prepareCall("{CALL demoSp(? , ?)}") ; 
			
	4. 执行SQL语句
		* Statement 接口提供了三种执行SQL语句的方法：executeQuery 、executeUpdate和execute
		* executeQuery 用来执行查询数据库的sql语句，返回ResultSet对象
		* execute 用于执行返回多个结果集、多个更新计数或二者组合的语句。
	
	5. 遍历结果集
		这里分为两种情况
		1. 这是一个增删改操作，那么将会返回本次操作影响的记录数
		2. 这是一个查询操作，那么返回结果将是一个ResultSet对象，遍历操作如下
			while(rs.next()){    
				String name = rs.getString("name") ;    
				String pass = rs.getString(1) ; // 此方法比较高效    
			}    
			
	6. 关闭JDBC对象资源
		* 注意关闭顺序，先关闭结果集 ResultSet，再关闭Statment，最后关闭连接
		

### 于是我们初步封装的jdbcUtil工具类如下
#### ```
public class JDBCUtil {
	private static ThreadLocal<Connection> tol = new ThreadLocal<>();

	private static Connection connection = null;
	private static PreparedStatement statement = null;
	private static ResultSet resultSet = null;

	private static String drive = "";
	private static String url = "";
	private static String uname = "";
	private static String pwd = "";

	private static Properties pp = null;
	private static InputStream fis = null;

	static {
		try {
			pp = new Properties();
			fis = JDBCUtil.class.getClassLoader().getResourceAsStream("db.properties");
			pp.load(fis);
			drive = pp.getProperty("drive");
			url = pp.getProperty("url");
			uname = pp.getProperty("username");
			pwd = pp.getProperty("password");
			Class.forName(drive);
		} catch (Exception e) {
			e.getMessage();
		}finally {
			try {
				fis.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

	public static Connection getConnection() {
		connection = tol.get();
		try {
			if(connection == null) {
				connection = DriverManager.getConnection(url, uname, pwd);
			}
		} catch (Exception e) {
			e.getMessage();
		}
		return connection;
	}

	public static void relese(Connection connection, PreparedStatement statement, ResultSet resultSet) {
		if (resultSet != null) {
			try {
				resultSet.close();
			} catch (Exception e) {
				e.getMessage();
			}
			resultSet = null;
		}
		if (statement != null) {
			try {
				statement.close();
			} catch (Exception e) {
				e.getMessage();
			}
			statement = null;
		}
		if (connection != null) {
			try {
				connection.close();
				tol.remove();
			} catch (Exception e) {
				e.getMessage();
			}
			connection = null;
		}
	}


}```