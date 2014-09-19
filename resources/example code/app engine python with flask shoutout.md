# Shoutout Demo - an App Engine Tutorial (with Java)

First, follow [the installation instructions](FAQ.md#google-app-engine-tutorial) to install Eclipse, and the Google plugin for Eclipse. At this point, you should have a newly created *Hello World* project.

Let's start by creating a ```/src/main/Shout.java``` class to represent a single Shout:
```java
package main;

import javax.jdo.annotations.IdentityType;
import javax.jdo.annotations.PersistenceCapable;
import javax.jdo.annotations.Persistent;
import javax.jdo.annotations.PrimaryKey;

@PersistenceCapable(identityType = IdentityType.APPLICATION)
public class Shout {
    @Persistent
    @PrimaryKey
    private String name;
    @Persistent
    private String message;
    
    public Shout(String name, String message){
        this.name = name;
        this.message = message;
    }
    
    public String getMessage() {
        return message;
    }
    public String getName(){
        return name;
    }
}
```

And create a typical [Persistence Manager Factory](https://db.apache.org/jdo/pmf.html) class at ```/src/main/PMF.java``` in order to use the database (via JDO):
```java
package main;

import javax.jdo.JDOHelper;
import javax.jdo.PersistenceManagerFactory;

public final class PMF {
    
    private static final PersistenceManagerFactory pmf = JDOHelper.getPersistenceManagerFactory("transactions-optional");
    
    private PMF(){}
    
    public static PersistenceManagerFactory get(){
        return pmf;
    }
}
```

Next, we'll create a ```/src/main/GAEDemoServlet.java``` servlet to handle GET requests that add new Shouts:
```java
package main;

import java.io.IOException;

import javax.jdo.PersistenceManager;
import javax.servlet.http.*;

@SuppressWarnings("serial")
public class GAEDemoServlet extends HttpServlet {
    public void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws IOException {
        PersistenceManager pm = PMF.get().getPersistenceManager();
        String message = req.getParameter("message");
        String name = req.getParameter("name");
        if ((name!=null)&&(message!=null)){
            try{
                Shout s = new Shout(name,message);
                pm.makePersistent(s);
            }finally{
                pm.close();
            }
        }
        resp.sendRedirect("/");

    }
}
```

Instead of using the existing ```/war/index.html```, we'll create the following ```/war/index.jsp```. This page will include a *form* to input new shouts, and will display all the existing shouts :
```jsp
<%@ page language="java" contentType="text/html; charset=windows-1255"
    pageEncoding="windows-1255"%>
<%@ page import="javax.jdo.PersistenceManager"%>
<%@ page import="javax.jdo.Query"%>
<%@ page import="java.util.List"%>
<%@ page import="main.*"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type"
    content="text/html; charset=windows-1255">
<title>Shoutout</title>
</head>
<body>
    <form action="gaedemo">
        name: <input type="text" name="name" /> 
        message: <input type="text" name="message"/>
        <input type="submit" value="Submit"/>
    </form>
    <%
    PersistenceManager pm = PMF.get().getPersistenceManager();
    Query query = pm.newQuery("SELECT FROM "+ Shout.class.getName() + " ORDER BY name DESC");
    List<Shout> entries = (List<Shout>) query.execute();
    for (int i=0;i<entries.size();i++){
        %><p><%=entries.get(i).getMessage()%> by <%=entries.get(i).getName()%></p><%
    }
    pm.close();
    %>
</body>
</html>
```

Lastly, we need to configure the [deployment descriptor](https://developers.google.com/appengine/docs/java/config/webxml) at ```/war/WEB-INF/web.xml``` to determine the URL mapping of our application:
```xml
<?xml version="1.0" encoding="utf-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns="http://java.sun.com/xml/ns/javaee"
xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" version="2.5">
    <servlet>
        <servlet-name>GAEDemo</servlet-name>
        <servlet-class>main.GAEDemoServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>GAEDemo</servlet-name>
        <url-pattern>/gaedemo</url-pattern>
    </servlet-mapping>
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```