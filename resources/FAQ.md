# Frequently Asked Questions

## Google App Engine Tutorial
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