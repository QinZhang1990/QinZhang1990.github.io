---
layout: post
title: "MySQL数据库分页"
date: 2017-09-23 23:30:00.000000000 +00:00
tags: 他山之石
---

## 1 分页的必要性

(1) 当数据量很大时，页面上将其全部显示出来，没有实际意义

(2) 遍历查询全部记录需要较多时间，占用较多带宽，响应客户端请求时间变长，效率降低，进而使用用户体验差

## 2 MySQL分页原理

(1) 数据库分页需要确定以下信息：

` pageSize `——每页显示的记录数据

` totalCount `——总共的记录数

` currentPage `——当前页数

` totalPage `——总共的页数，总页数需要向上取整，其计算公式如下：

totalPage = totalCount%pageSize==0? totalCount/pageSize:(totalCount/pageSize)+1;


(2) mysql数据库分页采用limit关键字

Select * from tableName limit m，n；

其中`m`是指记录开始的index，从0开始，表示第一条记录

`n`是指从第m+1条开始，取n条。

实际的分页中m = (currentPage-1)*pageSize; n = pageSize，如每页显示5条记录，现在查询第二页，则m=(2-1)*5，n=5；

## 3 代码实现

(1) 创建分页工具类pageUtil

```swift
/*注意此处需要导入包*/
public class PageUtil {
	private int pageSize;  //每页显示的条目数
	private int totalCount; //总条数
	private int currentPage; //当前页数
	private int totalPage;  //总页数
	public int getPageSize() {
		return pageSize;
	}
	public void setPageSize(int pageSize) {
		this.pageSize = pageSize;
	}
	public int getTotalCount() {
		return totalCount;
	}
	public void setTotalCount(int totalCount) {
		this.totalCount = totalCount;
	}
	public int getCurrentPage() {
		return currentPage;
	}
	public void setCurrentPage(int currentPage) {
		this.currentPage =currentPage;
	}
	public int getTotalPage() {
		return totalPage = totalCount%pageSize==0? totalCount/pageSize:(totalCount/pageSize)+1;
	}
	public void setTotalPage(int totalPage) {
		this.totalPage = totalPage; 
	}
}
```
(2) 创建BaseDao

```swift
/*注意此处需要导入包*/
public class BaseDao {
	private Connection conn; //建立连接
	private PreparedStatement ps;//sql语句
	private ResultSet rs;//返回的查询结果集
	
	private void getConnection(){//建立连接的方法
		try {
			//加载MySQL数据库驱动
			//不用记，Web App Libraries——mysql-connector——com.mysql.jdbc——Driver.class——右键单击——copy qualified name 
			Class.forName("com.mysql.jdbc.Driver");
			
			//数据库地址，用户名和密码
			String url = "jdbc:mysql://localhost:3306/booksys";
			String userName = "root";
			String pword = "123456";
			
			//建立连接，项目连接到本地的mysql数据库
			conn = DriverManager.getConnection(url,userName,pword);
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	//关闭 注意在给以下三个流加try/catch时 要分别加，不能整体上只加一个
	public void close(){
		if(rs != null)
			try {
				rs.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		if(ps!=null)
			try {
				ps.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		if(conn!=null)
			try {
				conn.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
	}
	//更新--增加、修改、删除
	//sql = insert into book(name,price,author,pubDate) values(?,?,?,?)
	public int executeUpdate(String sql,Object...objects){//三个点表示可变参数，第一个Object为对象类型
		try {
			this.getConnection();//建立数据库连接
			ps = conn.prepareStatement(sql);//构建动态的sql对象
			if(objects!=null){//设置参数
				for(int i=0;i<objects.length;i++){
					ps.setObject(i+1, objects[i]);//参数的索引从1开始
				}
			}
			return ps.executeUpdate();
		} catch (SQLException e) {
			e.printStackTrace();
		}finally{
			this.close();//关闭，进而释放资源
		}
		return -1;		
	}
	
	//查询
	public ResultSet executeQuery(String sql,Object...objects){
		try {
			this.getConnection();
			ps = conn.prepareStatement(sql);
			if(objects!=null){//设置参数
				for(int i=0;i<objects.length;i++){
					ps.setObject(i+1, objects[i]);
				}
			}
			return rs = ps.executeQuery();//查询得到的结果集
		} catch (SQLException e) {
			e.printStackTrace();
		}//此处不能关闭，一旦关闭rs中的返回查询结果就没有了
		return null;
	}
}
```

(3) 创建业务Dao层，并继承BaseDao

```swift
/*注意此处需要导入包*/
public class BookDao extends BaseDao {
	//查询所有书籍
	public List<Book> getAll(PageUtil p){
		List<Book> list = new ArrayList<Book>();
		String sql = "select * from book limit ?,?";
		//查询书籍的操作
		int currentPage = p.getCurrentPage();
		int pageSize = p.getPageSize();
		try {
			ResultSet rs = this.executeQuery(sql,(currentPage-1)*pageSize,pageSize); 
			while(rs.next()){
				list.add(new Book(rs.getInt(1),rs.getString(2),rs.getDouble(3),
						rs.getString(4),rs.getDate(5),rs.getInt(6)));
			}
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			this.close();
		}
		return list;		
	}
}
```
(4) 业务层分页的具体实现方法

```swift
private void list(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		PageUtil pUtil = new PageUtil();
		//设置pageSize
		pUtil.setPageSize(3);
		//设置当前页数
		int currentPage = 1;
		if(req.getParameter("currentPage")!=null){
			currentPage = Integer.parseInt(req.getParameter("currentPage"));
		}
		pUtil.setCurrentPage(currentPage);
		//设置总共记录数
		pUtil.setTotalCount(bookDao.totalCount());
		
		List<Book> list = bookDao.getAll(pUtil);
		List<Category> clist =categoryDao.getList();
		req.setAttribute("list", list);
		req.setAttribute("clist", clist);
		//将pageUtil放到前台页面，以供前台页面取其中的数据
		req.setAttribute("page", pUtil);
		req.getRequestDispatcher("list.jsp").forward(req, resp);
	}
```
注：参考了51cto学院邹波老师的课程[Java Web详解之JSP/Servlet视频课程]( http://edu.51cto.com/course/5023.html)


