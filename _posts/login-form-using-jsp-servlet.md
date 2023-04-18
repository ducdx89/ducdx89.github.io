---
layout: post
title: Tạo form login sử dụng JSP Servlet
subtitle: Servlet + JSP + JDBC + MySQL Example
cover-img: 
thumbnail-img: 
share-img: 
tags: [jsp, Servlet]
---

# Login Form using JSP + Servlet + JDBC + MySQL Example

author: [Duc SaDu](https://anvyshops.com "author profile")

  

In this article, we will build a simple Login Form using  [JSP],  [Servlet](, [JDBC]  and  [MySQL]  database.  
  

In this example, we will create an  **Employee Login Form**  and we will validate employee  _username_  and  _password_  with the database.  

# Video

This tutorial is explained in the below Youtube Video. Subscribe to my youtube channel at [Java Guides - YouTube Channel](https://www.youtube.com/channel/UC1Be9fnFTlcsUlejgfqag0g).  

  

Get source code of this tutorial on my  [**GitHub Repository**](https://github.com/RameshMF/servlet-tutorial).

  
Below diagram shows our Employee Login Form build using JSP:  

![](https://1.bp.blogspot.com/-yD4GjT1839M/XHokmceN0TI/AAAAAAAAFsA/oR8HaR2o01kBDzxA2SgN_SAmTEUHBh_vQCLcBGAs/s1600/jsp-servlet-login-form-example.PNG)

You can download the source code of this article from my GitHub repository. The link has given at end of this article.  
  
Let me list out the tools and technologies that I have used to develop this application.

## Tools and technologies used

-   JSP - 2.2 +
-   IDE - STS/Eclipse Neon.3
-   JDK - 1.8 or later
-   Apache Tomcat - 8.5
-   JSTL - 1.2.1
-   Servlet API - 2.5
-   MySQL - mysql-connector-java-8.0.13.jar

## Development Steps

1.  Create Eclipse Dynamic web project
2.  Add Dependencies
3.  Project Structure
4.  MySQL Database Setup
5.  Create a JavaBean - Login.java
6.  Create a LoginDao.java
7.  Create a LoginServlet.java
8.  Create a login.jsp
9.  Create a loginsuccess.jsp
10.  Demo

## 1. Create an Eclipse Dynamic Web Project

To create a new dynamic Web project in Eclipse:

1. On the main menu select **File > New > Project....**

2. In the upcoming wizard choose **Web > Dynamic Web Project.**

![](https://3.bp.blogspot.com/-CeA278XIG4g/XDiv1ePWVlI/AAAAAAAAFZI/2mvCujNS2Co_gp1UqevtbZk0qhDsaa9DwCLcBGAs/s1600/create-web-proj-1.png)

  

  

3. Click **Next**.

4. Enter project name as "login-jsp-servlet-jdbc-example";  
5. Make sure that the target runtime is set to Apache Tomcat with the currently supported version.

## 2. Add Dependencies

Add the latest release of below jar files to the lib folder.

```
- jsp-api.2.3.1.jar
- servlet-api.2.3.jar
- mysql-connector-java-8.0.13.jar

```

In Eclipse, paste these JAR files to your project directory:  **WebContent/WEB-INF/lib**

## 3. Project Structure

Standard project structure for your reference -  

![](https://3.bp.blogspot.com/-yBmG58tmJgE/XHolTnQAONI/AAAAAAAAFsI/penZTmhnu4QSmuEEg7nJHyRw8zO0VPkTACLcBGAs/s1600/project-structure.PNG)

## 4. MySQL Database Setup

Let's create a database named "mysql_database" in MySQL. Now, create a login table using below DDL script:
```sql
CREATE TABLE `login` (
  `username` varchar(45) NOT NULL,
  `password` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
Here is an Insert SQL statement:
```sql
INSERT INTO `mysql_database`.`login` (`username`, `password`) VALUES ("Ramesh", "Ramesh");
```
## 5. Create a JavaBean - LoginBean.java

Let's create a  _LoginBean_  class which we will use in JSP action tags.
```java
package net.javaguides.login.bean;

import java.io.Serializable;

public class LoginBean implements Serializable {
    /**
 * 
 */
    private static final long serialVersionUID = 1 L;
    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```
## 6. Create a LoginDao.java

Let's create a LoginDao class which contains JDBC code to connect with MySQL database. Add the following code to a LoginDao class:
```java
package net.javaguides.login.database;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import net.javaguides.login.bean.LoginBean;

public class LoginDao {

    public boolean validate(LoginBean loginBean) throws ClassNotFoundException {
        boolean status = false;

        Class.forName("com.mysql.jdbc.Driver");

        try (Connection connection = DriverManager
            .getConnection("jdbc:mysql://localhost:3306/mysql_database?useSSL=false", "root", "root");

            // Step 2:Create a statement using connection object
            PreparedStatement preparedStatement = connection
            .prepareStatement("select * from login where username = ? and password = ? ")) {
            preparedStatement.setString(1, loginBean.getUsername());
            preparedStatement.setString(2, loginBean.getPassword());

            System.out.println(preparedStatement);
            ResultSet rs = preparedStatement.executeQuery();
            status = rs.next();

        } catch (SQLException e) {
            // process sql exception
            printSQLException(e);
        }
        return status;
    }

    private void printSQLException(SQLException ex) {
        for (Throwable e: ex) {
            if (e instanceof SQLException) {
                e.printStackTrace(System.err);
                System.err.println("SQLState: " + ((SQLException) e).getSQLState());
                System.err.println("Error Code: " + ((SQLException) e).getErrorCode());
                System.err.println("Message: " + e.getMessage());
                Throwable t = ex.getCause();
                while (t != null) {
                    System.out.println("Cause: " + t);
                    t = t.getCause();
                }
            }
        }
    }
}
```
## 7. Create a LoginServlet.java


Let's create  _LoginServlet_  to process HTTP request parameters and redirect to the appropriate JSP page based on the employee login status. If login successfully validated with the database then redirect to  _loginsuccess.jsp_  page otherwise redirect to  _login.jsp_  page.  
```java
package net.javaguides.login.web;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import net.javaguides.login.bean.LoginBean;
import net.javaguides.login.database.LoginDao;

/**
 * @email Ramesh Fadatare
 */

@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    private static final long serialVersionUID = 1 L;
    private LoginDao loginDao;

    public void init() {
        loginDao = new LoginDao();
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

        String username = request.getParameter("username");
        String password = request.getParameter("password");
        LoginBean loginBean = new LoginBean();
        loginBean.setUsername(username);
        loginBean.setPassword(password);

        try {
            if (loginDao.validate(loginBean)) {
                //HttpSession session = request.getSession();
                // session.setAttribute("username",username);
                response.sendRedirect("loginsuccess.jsp");
            } else {
                HttpSession session = request.getSession();
                //session.setAttribute("user", username);
                //response.sendRedirect("login.jsp");
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
## 8. Create a login.jsp

Let's design login HTML form with following fields:

-   username
-   password
```html
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
 pageEncoding="ISO-8859-1"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="ISO-8859-1">
<title>Insert title here</title>
</head>
<body>
 <div align="center">
  <h1>Employee Login Form</h1>
  <form action="<%=request.getContextPath()%>/login" method="post">
   <table style="with: 100%">
    <tr>
     <td>UserName</td>
     <td><input type="text" name="username" /></td>
    </tr>
    <tr>
     <td>Password</td>
     <td><input type="password" name="password" /></td>
    </tr>

   </table>
   <input type="submit" value="Submit" />
  </form>
 </div>
</body>
</html>
```
## 9. Create a loginsuccess.jsp

After an employee successfully login then this page shows a successful message on screen:  
```html
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
 pageEncoding="ISO-8859-1"%>
<%@page import="net.javaguides.login.database.*"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="ISO-8859-1">
<title>Insert title here</title>
</head>
<body>
 <div align="center">
  <h1>You have logined successfully</h1>
 </div>
</body>
</html>
```
Note that in the above page, we have used JSP action tags. Read more about action tags here.

## 9. Demo

It's time to see a demo of the above development. Deploy this web application in tomcat server.

# Employee Login Form

Once you deploy this application successfully then hit this link into a browser - [http://localhost:8080/login-jsp-jdbc-mysql-example/login.jsp](http://localhost:8080/login-jsp-jdbc-mysql-example/login.jsp)

![](https://1.bp.blogspot.com/-yD4GjT1839M/XHokmceN0TI/AAAAAAAAFsE/GI94c2S9nOMTz6M3X4mf7TXFbBlmqd4wwCEwYBhgL/s1600/jsp-servlet-login-form-example.PNG)

### Login Success Page

![](https://3.bp.blogspot.com/-7RxQbHZxujs/XHongL1WJNI/AAAAAAAAFsU/khD8xUOlHBc48VDJlJEFuzObn0VxMXuAACLcBGAs/s1600/login-success-page.PNG)