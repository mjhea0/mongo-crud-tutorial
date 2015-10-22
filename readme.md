### Express Generator

`npm install -g express generator`

`brew install httpie` - we'll use this later

### Initital Project Structure 

>branch first-generator


├── app.js
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.jade
    ├── index.jade
    └── layout.jade

### Refactor with Swig

> branch second-swig

Remove jade templating and use swig instead.

Add this to your package.json
`"swig": "^1.4.2"`
Run npm install

In app.js remove:

``` javascript
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');
```


In app.js add:

```	javascript
var swig = require('swig')

var swig = new swig.Swig();
app.engine('html', swig.renderFile);
app.set('view engine', 'html');
```

In the views folder remove the all files and add a 
		layout.html
		index.html

Cut and paste this code into the layout.html:

``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>{{ title }}</title>
    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
    <link rel="stylesheet" href="/css/main.css">
  </head>
  <body>
    {% block content %}
    {% endblock %}
    <script type="text/javascript" src="//code.jquery.com/jquery-2.1.4.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
    <script type="text/javascript" src="/js/main.js"></script>
  </body>
</html>

```

Cut and paste this into you index.html:

```
{% extends 'layout.html' %}

{% block title %}{% endblock %}


{% block content %}

  <div class="container">

    <h1>{{ title }}</h1>
    <p>Welcome to {{ title }}</p>

  </div>

{% endblock %}

```

Now in the terminal, in the root of the project directory run: 
'npm start' 

If everything is setup correctly you should see this:

** img of express page

### Set Up MongoDB with Mongoose

> branch third-mongodb

Start by installing Mongo and Mongoose:

`npm install mongodb -g`

`npm install mongoose --save`

Have a look at the package.json file. We now have mongoose in our dependencies.

Lets look at the MongoDB through our terminal.
In the command line type `sudo mongod`, then  enter you password. This will start the MongoDB daemon running. Now open a new tab and type `mongo`. This is the command line interface for mongoDB. Enter the command `show dbs`. This will show any databases that you have created. If you are using mongo for the first time it should be !?!?!?!?!.


Next add a new file, 'database.js' to the root directory.

In database.js add the following code:

```
// bring in mongoose and grab Schema constructor
var mongoose = require('mongoose');
var Schema = mongoose.Schema;


// create new Schema, setting keys and value types
var itemSchema = new Schema ({
	name: String,
	type: String
});

// create a model, which is a Mongo collection, which is akin to a SQL table
var Item = mongoose.model('items', itemSchema);

// set up the connection to the local database, if it doesn't exist yet one will be created automatically
mongoose.connect('mongodb://localhost/mongo-item');

// make the Item Schema available to other files
module.exports = Item;


```
Possible schema field value types:

* String
* Number
* Date
* Buffer
* Boolean
* Mixed
* Objectid
* Array

http://mongoosejs.com/docs/schematypes.html


talk about ObjectId at some point

### CRUD Routes

What is CRUD?

* Create
* Read
* Update
* Delete

These are the basic operations that an app needs to perform when handling data. You may have heard of a RESTful API, which is similar, but with specific philosiphies applied that are out of the scope of this tutorial. For more information go [here](http://www.restapitutorial.com/lessons/whatisrest.html). As a programmer, if you can elegantly handle these actions then you are well on your way to turning your skills into a paycheck. 

Let's begin with our index.js file.

In index.js we need to require our database file to get access to the Schema:

``` javascript
var Item = require('../database.js');
```

The standard setup that our express generator provided has 'express' required in our `index.js` and then sets the variable `router` to an instance of an express router object. This object will handle the transfering/serving of data as called for by our HTTP requests. The router object includes functions that we can call to acheive our basic CRUD operations. When we define these CRUD operations using the router instance, we are creating routes. I think of them as pathways for data between our browser/server and the database. So, lets get to it.

#### READ

We need to be able to read or get all of the Items from the database.

```
// call the GET method, and define an anonymous function
router.get('/items', function(req, res, next) {

// instantiate a new Item with the values supplied by the request  
	Item.find({}, function(err, data){

// handle an error
		if (err) {
			res.json(err);
		}
// handle an empty database 
		else if (data.length===0) {
			res.json({message: 'There are no items in the database.'});
		}
// if there are Items, return them	
		else {
			res.json(data);
		}
	});
});
```

There is a TON going on in there and if you are new to routes, it is really intimidating. I broke it down it down somewhat in the comments, but we can dig deeper. 

After we call `router.get` we define the URL path, in this case `/items`. Our app will use this 'location' or 'address' to utilize the `GET` functionallity of the app.

Let me show you what I mean with that 'httpie' we installed earlier. In the terminal fire up the database using `sudo mongod`, and also the server using `npm start`. Now run:

```

http GET localhost:3000/items

```

You should see this:

image httpie-get-no-items

You can see in the second line `HTTP/1.1 200 OK`. This means our route was successful and that the logic in our route was executed. You can confirm that this logic was correct becuase it returned "There are no Items in the database." in 'json' format. We'll come back and test this some more after we create some Items. 


First, lets Change the route path to illustrate a point. Run:

```

http GET localhost:3000/things

```

You should see this: 

image httpie-get-404

We went to an undefined route, so there was nothing for the browser/server to do. This is a 404 error and they are common in devopling and tell you that you need to investigate your routes.

OK, back to the CRUD.

#### CREATE

We need to be able to create information and add or `POST` it to our database. 

```javascript
// call the post method, and define an anonymous function
router.post('/items', function(req, res, next) {

// instantiate a new Item with the values supplied by the request  
	var newItem = new Item({name: req.body.name, type: req.body.type});

// save the new item using a mongoose function
	newItem.save(function(err, data){
// handle an error 
		if (err) {
			res.json(err);
		}
// no error, then return the data in the json format
		else {
			res.json(data);
		}
	});
});

```

Have a look at the `req.body`. 'req' stands for 'request'. It is an object sent by the browser/server with properties that we can access. We are grabbing the 'body' property and getting its values to instantiate our new Item.

Now we will test this out with httpie in the terminal. Run:

```
http -f POST localhost:3000/items name="bicycle" type="vehicle"

```

You should see this:

image httpie-post-200


Ok, awesome, we can create new Items. Lets look a little closer at that json object that came back. It has a 'name' and 'type' property which we should expect. But it also has '_id' and '_v'. We won't worry about the latter in this tutorial, but I do want to look at the former.

'_id' is a unique id created by Mongo when a new item is saved. It will always be unique, always. This is important and it is extremely useful.  A couple use cases are finding documents in your database, or diferentiating between two similar documents.  

We are going to put it to work in our 'update' route.

#### UPDATE

```

```


#### DELETE

```
router.delete('/items/:id', function(req, res, next) {
	Item.findOneAndRemove({_id: req.params.id}, function(err, data){
		if (err) {
			res.json(err.message);
		}
		else if (data.length===0) {
			res.json({message: 'An item with that id does not exist in this database.'});
		}
		else {
			res.json({message: 'Success. Item deleted.'});
		}
	});
});

```





















