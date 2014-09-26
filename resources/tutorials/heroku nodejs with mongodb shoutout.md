# Shoutout Demo - Heroku Tutorial (with Node.js and MongoDB)

### Initial Heroku Setup
First, create a [free account on Heroku](https://signup.heroku.com/signup/dc).

Make sure you have [Node.js](http://nodejs.org/) and [npm](https://github.com/npm/npm#synopsis) installed.

Download and install [the Heroku Toolbelt](https://devcenter.heroku.com/articles/getting-started-with-python#set-up). Once installed, you can use the ```heroku login``` command to connect to your Heroku account.

### Creating the Shoutout Application

In this tutorial we will use the NoSQL database called [MongoDB](http://www.mongodb.org/). Download and install MongoDB (installation instructions can be found on the [MongoDB website](http://www.mongodb.org/downloads)).

Create a new directory for your application:
```bash
mkdir shoutout
cd shoutout
```

Install the following packages with ```npm```:
```bash
npm install express
npm install body-parser
npm install ejs
npm install mongoose
```

Generate the ```package.json``` file by using:
```bash
npm init
```
The created file should look similar to this:
```json
{
  "name": "shoutout",
  "version": "0.0.1",
  "description": "shoutout demo",
  "scripts": {
    "start": "node main.js"
  },
  "author": "",
  "license": "MIT",
  "dependencies": {
    "body-parser": "^1.8.2",
    "express": "^4.9.3",
    "mongoose": "^3.8.16",
    "ejs": "^1.0.0"
  },
  "engines": {
    "node": "0.10.x"
  },
  "main": "main.js",
  "devDependencies": {}
}
```

Create and assign a Heroku application ID. We will ask Heroku to create an application ID of ```shoutoutdemo``` (assuming it is available):
```bash
git init
heroku create shoutoutdemo
```

Add the MongoHQ addon to your Heroku application:
```bash
heroku addons:add mongohq
```
**Note, there is a need to verify your billing information on Heroku in order to use addons**, even though MongoHQ is free to use for 512mb ([a possible workaround](http://www.elliotbradbury.com/use-mongohq-heroku-without-verifying-account/)).

<br>  
Next, we'll create the backend for our application by creating ```main.js```, to handle POST requests that add new shouts, and GET requests that display existing shouts:
```js
var express = require('express');
var mongoose = require('mongoose');
var bodyParser = require('body-parser');

mongoose.connect(process.env.MONGOHQ_URL || 'mongodb://localhost/shoutout');

// define model
var Shout = mongoose.model('Shout', { name: String, message: String });

// setup middleware
var app = express();
app.use(bodyParser.urlencoded({ extended: false }))
app.engine('html', require('ejs').renderFile);
app.set('view engine', 'html');


app.get('/', function (req,res){
    Shout.find(function(err, shouts) {
        if (err)
            return console.error(err);
        res.render('index.html', { shouts : shouts });
    });
});

app.post('/post', function (req, res) {

    var name = req.body.name;
    var message = req.body.message;
    var shout = new Shout({ name: name, message: message });
    shout.save(function(err) {
        if (err)
            return console.error(err);
    });
    res.redirect(301, '/');
});

var port = process.env.PORT || 3000;
app.listen(port);
console.log('started on port', port);
```

Lastly, we'll create the frontend of our application - the ```views/index.html``` template file. This page will include a *form* to input new shouts, and will display all the existing shouts. The templating engine we use is [ejs](https://github.com/visionmedia/ejs) ([jade](http://jade-lang.com/) is an alternative templating engine):
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
    <% shouts.forEach(function(shout){ %>
    <%= shout.name %> - <%= shout.message %> <br>
    <% }); %>
</body>
</html>
```

Lastly, we need to create a ```Procfile``` that contains:
```
web: node main.js
```

Deploy to Heroku and test:
```bash
git add main.js packages.json Procfile views/
git commit -m "initial commit"
git push heroku master
heroku open
```

We can see the live application running on Heroku at ```http://<app id>.herokuapp.com```, in our case it would be ```http://shoutoutdemo.herokuapp.com```. We can manage our application with the [Heroku dashboard](https://dashboard-next.heroku.com/apps).

### How do I continue from here?

For beginners with Node.js, you can watch the following YouTube tutorial for Node.js and [Express.js](http://expressjs.com/).
[![Node.js & Express 101](http://img.youtube.com/vi/BN0JlMZCtNU/0.jpg)](http://www.youtube.com/watch?v=BN0JlMZCtNU)

You should also check out [this webinar](https://www.youtube.com/watch?v=xuXIBSa_7j4), which shows how to use [WebStorm](http://www.jetbrains.com/webstorm/) (FREE for students) with Node.js and Heroku.