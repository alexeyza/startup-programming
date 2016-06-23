# Shoutout Demo - App Engine Tutorial (with Python and Flask)

In this tutorial, we'll start by following the YouTube tutorial below, in order to set the initial working environment. By following the instructions, you should have a working *Hello World* App Engine/Flask project.

[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/FRI3QGNWJYI/0.jpg)](http://www.youtube.com/watch?v=FRI3QGNWJYI)

Let's start by creating a ```models.py``` module with a Shout class to represent a single Shout:
```python
from google.appengine.ext import ndb


class Shout(ndb.Model):
    name = ndb.StringProperty(required=True)
    message = ndb.TextProperty(required=True, indexed=False)
```

Next, we'll create the backend for our application at ```main.py```, to handle POST requests that add new shouts, and GET requests that display existing shouts:
```python
import sys
from models import Shout
import os
sys.path.insert(1, os.path.join(os.path.abspath('.'), 'venv/lib/python2.7/site-packages'))
from flask import Flask, render_template, request, redirect

app = Flask(__name__)

@app.route("/", methods=['GET'])
def index():
    shouts = Shout.query()
    return render_template('index.html', shouts=shouts)

@app.route("/post", methods=['POST'])
def post():
    s = Shout(name=request.form['name'], message=request.form['message'])
    s.put()
    return redirect('/')

if __name__ == "__main__":
    app.run()
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

Note, that in order to deploy your application to App Engine, you need to make sure your **application ID** is configured correctly at ```app.yaml``` as follows:
```
application: appenginedemo
version: 1
runtime: python27
api_version: 1
threadsafe: yes

handlers:
- url: /favicon\.ico
  static_files: favicon.ico
  upload: favicon\.ico

- url: .*
  script: main.app

libraries:
- name: webapp2
  version: "2.5.2"

```

### How do I continue from here?
[Flask](http://flask.pocoo.org/) is a very simple web framework for Python. There are several good resources to learn Flask:

- [Official documentation](http://flask.pocoo.org/docs/)
- [Flask by Example](https://www.youtube.com/watch?v=FGrIyBDQLPg) is a great YouTube tutorial, with all the [code availible online](https://github.com/miguelgrinberg/flask-pycon2014) as well
- [Explore Flask](http://exploreflask.com/) online book

Possible alternatives to Flask are the App Engine default [webapp2](https://cloud.google.com/appengine/docs/python/gettingstartedpython27/usingwebapp) and the popular [Django](https://www.djangoproject.com/).
