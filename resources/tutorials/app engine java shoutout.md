# Shoutout Demo - App Engine Tutorial (with Java)

### Initial Google App Engine Setup
The following instructions will help you use Google App Engine with Java.

1. Install Eclipse (choose the [Eclipse IDE for Java EE Developers](https://www.eclipse.org/downloads/packages/eclipse-ide-java-ee-developers/lunar) version).
2. Make sure the version of Java on your system is Java 7 (currently Java 8 causes issues with the Google Plugin for Eclipse).
3. Install the Google plugin for Eclipse by following [these instructions](https://developers.google.com/appengine/docs/java/tools/eclipse). Make sure to choose the version of Eclipse you have installed.
4. During the installation of the Google plugin for Eclipse, you will be asked to choose which components you would like to install. If you only want to use App Engine (without GWT and without Android), check the "**Google plugin for Eclipse (required)**" checkbox.
    ![Alt text](https://developers.google.com/eclipse/images/luna-install.png)
5. Create a new project for App Engine 
    - ```New -> Project -> Google -> Web Application Project``` 
    - Give name to your project and the name of the default package (e.g. controller).
    - Uncheck "**Use Google Web Toolkit**" if you do not intend to use it or haven't installed it with the Google Plugin for eclipse.
    - Right click the project and choose: ```Run as -> web application```.
6. The steps above allow you to run the project locally (on your machine).
    - Open [http://localhost:8888/](http://localhost:8888/) in your browser for your running application.
    - Open [http://localhost:8888/_ah/admin](http://localhost:8888/_ah/admin) to manage your local application (i.e. to manipulate the DB).
7. Next we will configure Eclipse to work with your App Engine account.
8. Open the [App Engine website](https://appengine.google.com/), login, and create a new  application. Remember the unique **application ID** you are giving that application. You will need to configure it in Eclipse as well.
9. In eclipse, right click you project and then choose: ```Properties -> Google -> App engine``` and set the **application ID** you've chosen earlier.
10. Deploying to App Engine:
    - Right click on the project and choose ```Google -> Deploy to App Engine```
    - You will be prompted to approve Eclipse to access your Google account.
    - Follow the instructions and click ```Deploy```
    - Wait until it finishes deployment.
    - Open the following URL: ```<your_application_id>.appspot.com``` to see your application running on App Engine (wait a few seconds for App Engine to start an instance).
    - Manage your application at [the App Engine website](https://appengine.google.com/).

### Creating the Shoutout Application

At this point, you should have a newly created *Hello World* project.

Let's start by creating a ```src/main/Shout.java``` class to represent a single Shout:
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

And create a typical [Persistence Manager Factory](https://db.apache.org/jdo/pmf.html) class at ```src/main/PMF.java``` in order to use the database (via JDO):
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

Next, we'll create a ```src/main/GAEDemoServlet.java``` servlet to handle GET requests that add new shouts:
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

Instead of using the existing ```war/index.html```, we'll create the following ```war/index.jsp```. This page will include a *form* to input new shouts, and will display all the existing shouts:
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

Lastly, we need to configure the [deployment descriptor](https://developers.google.com/appengine/docs/java/config/webxml) at ```war/WEB-INF/web.xml``` to determine the URL mapping of our application:
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