---
layout : post
title : java 连接 access 数据库
category  : "java"
tags : [java,database,access]
---

Java 连接access需要安装和jdk位数相对应的office，然后安装对应位数的[access driver](http://www.microsoft.com/en-us/download/confirmation.aspx?id=13255)(这里最好下载英文版的，之前下载的中文版居然无法安装，不知道什么问题)。

>`jdk x64` --> `office X64` --> `access manager X64`

>`jdk x86` --> `office x86` --> `access manager x86`

连接代码

{% highlight java %}
package com.cyberdak.util.access;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;

import sy.model.access.ReAdd;

/**
 * @author 亢伟楠
 *	时间:2015年3月9日下午4:23:48
 */
public class AccessDBUtil {
	
	static String accessFile = "d:\\a.accdb";
	
	public static void main(String[] args) throws SQLException {
		getReAddList();
	}

	public static  List<ReAdd> getReAddList() throws SQLException{
		Connection conn =  null;
		final String sql = "select * from `追加总表`";
		ResultSet rs = null;
		List<ReAdd> list = null;
		try{
			conn = getConnection();
			rs = conn.createStatement().executeQuery(sql);;
			while(rs.next()){
				ReAdd  reAdd = new ReAdd();
				reAdd.setColor(rs.getString("颜色"));
				System.out.println(reAdd.getColor());
				list.add(reAdd);
			}
		}finally{
			rs.close();
			conn.close();
		}
		return list;
	}

	public static Connection getConnection() throws SQLException{  
		Properties prop = new Properties();     
		prop.put("charSet", "gb2312");  //设置编码防止中文出现乱码  
		return DriverManager.getConnection("jdbc:odbc:Driver={MicroSoft Access Driver (*.mdb, *.accdb)};DBQ="+accessFile, prop);
	}


}



{% endhighlight %}

连接的第二个参数切记要设置,不然连报错信息都是乱码。