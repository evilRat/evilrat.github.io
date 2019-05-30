---
layout: post
title:  "数据库和TXT文件内容的交换"
date:   2016-07-07
excerpt: "IO&数据库练习"
project: true
tag:
- oop
comments: true
---



### 前言

今天做了一个简单的实验，通过这次试验来练习IO流，数据库连接，SQL等知识。下面是详细信息。

### 项目结构

<img src="/assets/img/flant.png">




### 代码

#### bean.Content

```

package bean;
public class Content {
	private int id;
	private String content;
	public Content(int id,String content){
		this.id=id;
		this.content=content;
	}	
	public void setId(int id){
		this.id=id;
	}
	public int getId(){
		return id;
	}
	public void setContent(String content){
		this.content=content;
	}
	public String getContent(){
		return content;
	}
	public String toString(){	
		return id+","+content;	
	}	
}


```

#### bean.Student

```

package bean;

public class Student {
	private int id;
	private String name;
	private String sex;
	private int age;
	public Student(int id,String name,String sex,int age){
		this.id=id;
		this.name=name;
		this.sex=sex;
		this.age=age;
	}	
	public void setId(int id) {
		this.id = id;
	}
	public int getId() {
		return id;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getName() {
		return name;
	}
	public void setSex(String sex) {
		this.sex = sex;
	}
	public String getSex() {
		return sex;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public int getAge() {
		return age;
	}
}


```

#### dao.ContentDao

```
package dao;

import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

import bean.Content;
import utils.DBUtil;
public class ContentDao {	
	DBUtil db=new DBUtil();
	PreparedStatement pre=null;
	ResultSet re=null;	
	public List<Content> getInfoFromDB(){
		db.getConnection();
		List<Content> list= new ArrayList<Content>();
		String sql="select * from content;";
		try {
			pre=db.getCon().prepareStatement(sql);
		} catch (SQLException e1) {
			// TODO Auto-generated catch block
			e1.printStackTrace();
		}
		try {
			re=pre.executeQuery();
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		try {
			while(re.next()){
				list.add(new Content(re.getInt(1),re.getString(2)));
			}
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return list;
	}
}

```

#### dao.Student

```

package dao;

import java.sql.SQLException;
import java.util.List;
import java.sql.PreparedStatement;
import bean.Student;
import utils.DBUtil;
public class StudentDao {	
	DBUtil db=new DBUtil();
	PreparedStatement pre=null;	
	public void senttoDB(List<Student> list){
		db.getConnection();
		for(Student st:list){
			String sql="insert into Student values(?,?,?,?)";
			try {				
				pre=db.getCon().prepareStatement(sql);
				pre.setInt(1, st.getId());
				pre.setString(2, st.getName());
				pre.setString(3, st.getSex());
				pre.setInt(4, st.getAge());						
				pre.executeUpdate();		
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}finally{										
			}
		}
		db.closeConnection(pre,null);
	}
}

```

#### test.Test

```

package test;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import bean.Content;
import bean.Student;
import dao.ContentDao;
import dao.StudentDao;

public class Test {
	public static void main(String[]args){
		
//		readFromFile();
		writetoFile();				
	}
	public static void readFromFile(){
		StudentDao stuDao=new StudentDao();
		String line;
		List<Student> list =new ArrayList<Student>();		
		File file=new File("files/read.txt");		
		try {
			FileReader in=new FileReader(file);
			BufferedReader read=new BufferedReader(in);
			while((line=read.readLine())!=null){
				String l[] =line.split(",");
				
//				System.out.println(line);
				list.add(new Student(Integer.parseInt(l[0]),l[1],l[2],Integer.parseInt(l[3])));
				
//				read.readLine();
			}
			stuDao.senttoDB(list);
			read.close();
			in.close();			
		} catch (IOException e) {
			System.out.println("not found this file");
			e.printStackTrace();
		}					
	}	
	public static void writetoFile(){		
		ContentDao contDao=new ContentDao();
		File file=new File("files/write.txt");
		try {
			BufferedWriter out=new BufferedWriter(new FileWriter(file));
//			List <Content> list=new ArrayList<Content>();
//			list=contDao.getInfoFromDB();			
			for(Content cont:contDao.getInfoFromDB()){
				System.out.print(cont.toString());
				out.write(cont.toString());
				out.newLine();
				out.flush();
			}
			out.close();											
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}						
	}
	
	
	
}

```

#### utils.DBUtil

```

package utils;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class DBUtil {	
	private Connection con=null;	
	public Connection getCon() {
		return con;
	}	
	public void getConnection(){						
		try {
			Class.forName("com.mysql.jdbc.Driver");
			con=DriverManager.getConnection("jdbc:mysql://localhost:3306/db_test","root","root");
//			String sql="insert into ";
//			pre=con.prepareStatement(sql);									
		} catch (ClassNotFoundException | SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}					
	}
	public void closeConnection(PreparedStatement pre,ResultSet re){		
		if(re!=null){
			try {
				re.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		if(pre!=null){
			try {
				pre.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		if(con!=null){
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

### 总结

通过这次试验，让我熟练了IO和数据库的一些知识，但是并不扎实，课余时间还要多练



























<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-StuSystem/" data-title="StudentSystem" data-url="http://kongzheng1993.github.io/kongzheng1993-StuSystem/"></div>
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