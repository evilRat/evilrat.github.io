---
layout: post
title:  "Mysql to Oracle"
date:   2016-07-19
excerpt: "实现mysql到oracle的转换"
project: true
tag:
- oop
comments: true
---

通过这次实验，我了解了连接oracle和mysql的区别，详情请访问我的[github](https://github.com/kongzheng1993/MysqlToOracle)

这里我只谈在编写中遇到的问题：

#### 连接方法：
+ Mysql

```

package com.isoft.mto.util;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
public class Mcon {	
	Connection con=null;	
	public void getConnection(){		
		try {
			Class.forName("com.mysql.jdbc.Driver");
			con=DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/sakila","root","root");						
		} catch (ClassNotFoundException | SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}		
	}
	public Connection getCon(){
		return con;
	}
	public void close(Connection con,PreparedStatement pre,ResultSet re){
		if(re!=null){
			try {
				re.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}			
		}if(pre!=null){
			try {
				pre.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}if(con!=null){
			try {
				con.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}	
	}	
}

```

+ Oracle

```

package com.isoft.mto.util;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
public class Ocon {
	private Connection con=null;		
	public Connection getCon(){
		return con;
	}
	public void close(Connection con,PreparedStatement pre,ResultSet re){
		if(re!=null){
			try {
				re.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}			
		}if(pre!=null){
			try {
				pre.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}if(con!=null){
			try {
				con.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}		
	}	
	public void getConnection(){
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");			
			con=DriverManager.getConnection("jdbc:oracle:thin:@127.0.0.1:1521:XE","hr","root");												
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			System.out.println("数据库连接出错！！！");
			e.printStackTrace();
		}				
	}	
}

```


#### Mysql和Oracle在jdbc中sql语句的区别

+ mysql可以传入“；”（可有可无，不影响）；
+ oracle不可以传入“；”（在sql字符串中加上“；”会导致sql语法错误）。

#### 关于两种DriverManager.getConnection();

+ mysql:"jdbc:mysql://127.0.0.1:3306/sakila","root","root"
+ oracle:"jdbc:oracle:thin:@127.0.0.1:1521:XE","hr","root"


#### 不足

这次试验数据库的信息都是已知的，并没有使用getTypeName(),getCollumName()等方法，所以有很多可以改进的地方。


















<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-mto/" data-title="mto" data-url="http://kongzheng1993.github.io/kongzheng1993-mto/"></div>
<script type="text/javascript">
var duoshuoQuery = {short_name:"kongzheng1993"};
	(function() {
		var ds = document.createElement('script');
		ds.type = 'text/javascript';ds.async = true;
		ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
		ds.charset = 'UTF-8';
		(document.getElementsByTagName('head')[0] 
		 || document.getElementsByTagName('body')[0]).appendChild(ds);
	})();
</script>
</html>