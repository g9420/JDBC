# JDBC 封装工具类
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

	public static void executeUpdate(String sql,Object[] params) {
		try {
			connection = JDBCUtil.getConnection();
			statement = connection.prepareStatement(sql);
			if(params != null && params.length != 0) {
				for (int s = 0; s < params.length; s++) {
					statement.setObject(s + 1, params[s]);
				}
			}
				statement.executeUpdate();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public static List<Map<String,Object>> exectueQuery(String sql,Object[] params){
		try {
			connection = JDBCUtil.getConnection();
			statement = connection.prepareStatement(sql);
			if(params != null && params.length != 0) {
				for (int s = 0; s < params.length; s++) {
					statement.setObject(s + 1, params[s]);
				}
			}
			resultSet = statement.executeQuery();
			ResultSetMetaData rsmd = (ResultSetMetaData) resultSet.getMetaData();
            int columnCount = rsmd.getColumnCount();
            ArrayList<Map<String, Object>> list = new ArrayList<>();
            while (resultSet.next()) {
				HashMap<String, Object> map = new HashMap<>();
				for(int i =0;i<columnCount;i++) {
					String columnName = rsmd.getColumnLabel(i+1);
					Object columnValue = resultSet.getObject(i+1);
					//System.out.println(columnName + "===>" + columnValue);
					map.put(columnName, columnValue);
				}
				
			}
            
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
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


}

