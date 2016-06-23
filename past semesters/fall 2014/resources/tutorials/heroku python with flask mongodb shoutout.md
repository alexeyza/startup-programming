# Shoutout Demo - Heroku Tutorial (with Python, MongoDB and Flask)

### Initial Heroku Setup
First, create a [free account on Heroku](https://signup.heroku.com/signup/dc).

Next, install Python, Pip and Virtualenv based on your operation system: [Windows](http://docs.python-guide.org/en/latest/starting/install/win/), [OS X](http://docs.python-guide.org/en/latest/starting/install/osx/), or [Linux](http://docs.python-guide.org/en/latest/starting/install/linux/).

Download and install [the Heroku Toolbelt](https://devcenter.heroku.com/articles/getting-started-with-python#set-up). Once installed, you can use the ```heroku login``` command to connect to your Heroku account.

### Creating An Initial Hello World Application

Create a new directory for your application:
```bash
mkdir shoutout
cd shoutout
```
Create a new virtual environment for your application:
```bash
virtualenv venv
```
Activate the virtual environment:
```bash
source venv/bin/activate
```
This allows to install new Python packages with ```pip``` in the virtual environment of the current project without affecting other projects. In order to deactivate (when you're done working on the project) use the ```deactivate``` command.

Install flask:
```bash
pip install flask
```

Create a basic backend for your application at ```main.py```:
```python
import os
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello world!"

if __name__ == "__main__":
    port = int(os.environ.get("PORT", 5000))
    app.run(host='0.0.0.0', port=port)
```

<br>
Now, there are two additional files required to deploy to Heroku: ```requirements.txt``` and ```Procfile```. The requirements file will include the package names needed for our application to run. We can generate it using the following command:
```bash
pip freeze > requirements.txt
```
This should create a ```requirements.txt``` file that is similar to this:
```
Flask==0.10.1
Jinja2==2.7.3
MarkupSafe==0.23
Werkzeug==0.9.6
argparse==1.2.1
itsdangerous==0.24
wsgiref==0.1.2

```

Next, create a file named ```Procfile``` to include the following:
```
web: python main.py
```

<br>
Now, let's deploy your *Hello World* application to Heroku to make sure everything works so far. We will ask Heroku to create an application ID of ```shoutoutdemo``` (assuming it is available). 
```bash
git init
git add main.py requirements.txt Procfile
git commit -m "initial commit"
heroku create shoutoutdemo
git push heroku master
heroku open
```
We can see the live application running on Heroku at ```http://<app id>.herokuapp.com```, in our case it would be ```http://shoutoutdemo.herokuapp.com```. We can manage our application with the [Heroku dashboard](https://dashboard-next.heroku.com/apps).

### Creating the Shoutout Application

In this tutorial we will use the NoSQL database called [MongoDB](http://www.mongodb.org/). Download and install MongoDB (installation instructions can be found on the [MongoDB website](http://www.mongodb.org/downloads)).

Install [pymongo](http://api.mongodb.org/python/current/):
```bash
pip install pymongo
```

Update the ```requirements.txt``` file to include pymongo:
```bash
pip freeze > requirements.txt
```

Add the MongoHQ addon to your Heroku application:
```bash
heroku addons:add mongohq
```
**Note, there is a need to verify your billing information on Heroku in order to use addons**, even though MongoHQ is free to use for 512mb ([a possible workaround](http://www.elliotbradbury.com/use-mongohq-heroku-without-verifying-account/)).

Find the ID of the database MongoHQ has assigned for your application with the following command:
```bash
heroku config
```
The output should be similar to this:
```
mongodb://<user>:<password>@kahana.mongohq.com:10087/app29843323
```
In which case, the database ID is the last part - ```app29843323```. We will use it to set the connection later.

<br>  
Next, we'll create the backend for our application by modifying ```main.py```, to handle POST requests that add new shouts, and GET requests that display existing shouts:
```python
import os
from flask import Flask, render_template, request, redirect
import pymongo
from pymongo import MongoClient


MONGO_URL = os.environ.get('MONGOHQ_URL')
client = MongoClient(MONGO_URL)

# Specify the database
db = client.app29843323
collection = db.shoutouts

app = Flask(__name__)

@app.route("/", methods=['GET'])
def index():
    shouts = collection.find()
    return render_template('index.html', shouts=shouts)

@app.route("/post", methods=['POST'])
def post():
    shout = {"name":request.form['name'], "message":request.form['message']}
    shout_id = collection.insert(shout)
    return redirect('/')


if __name__ == "__main__":
    port = int(os.environ.get("PORT", 5000))
    app.run(host='0.0.0.0', port=port)
```

Lastly, we'll create the frontend of our application - the ```templates/index.html``` template file. This page will include a *form* to input new shouts, and will display all the existing shouts:
```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>Shoutout</title>
</head>
<body>
    <form action="/post" method="post">
        name: <input type="text" name="name"/>
        message: <input type="text" name="message"/>
        <input type="submit" value="Submit"/>
    </form>
    {% for shout in shouts %}
    {{ shout.name }} - {{ shout.message }} <br>
    {% endfor %}
</body>
</html>
```

Deploy to Heroku and test:
```bash
git add main.py requirements.txt templates/
git commit -m "add MongoDB"
git push heroku master
heroku open
```

### How do I continue from here?
[Flask](http://flask.pocoo.org/) is a very simple web framework for Python. There are several good resources to learn Flask:

- [Official documentation](http://flask.pocoo.org/docs/)
- [Flask by Example](https://www.youtube.com/watch?v=FGrIyBDQLPg) is a great YouTube tutorial, with all the [code availible online](https://github.com/miguelgrinberg/flask-pycon2014) as well
- [Explore Flask](http://exploreflask.com/) online book

A popular alternative to Flask is [Django](https://www.djangoproject.com/).