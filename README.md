# Python and Flask Workshop
Welcome to the python-flask-app repo.  Today we will be building the image and running the container for **Python** and **Flask**, which can be found in this repo.  But in order to make this general app your own, we will need to learn more about **Github**.

* What's Github? - https://guides.github.com/activities/hello-world/
* What's Python? - https://www.w3schools.com/python/python_intro.asp
* What's Flask? - https://palletsprojects.com/p/flask/
   * Flask API: https://flask.palletsprojects.com/en/1.1.x/api/

You will use Flask to build a web application to interact with your MariaDB servier.

#### Prerequisites
1. Create a free github user account - https://github.com/join?plan=free&source=pricing-card-free
1. Install SublimeText for syntax coloring - https://www.sublimetext.com/

## Github Primer
1. Complete the Github hello-world activity linked to above: https://guides.github.com/activities/hello-world/ 

## Create your final project webapp
1. From the https://github.com/munners17/python-flask-app repo, click the *Use this template* button
1. Name the repo after a webapp name related to your final project
1. Choose a private repo!  
	*Note:  Feel free to make your project repo public after projects have been turned in*
1. Clone your newly created repo using your method of choice.

## Review the Dockerfile

## Build and Start Flask Container
1.  In the command line, move to your previously cloned repo
    * `cd path/to/repo`
1.  Build the docker image `docker build -t munners17/python-flask .`
1.  Create a docker network to allow different docker containers to easily communicate with each other over an IP network. Then bind the existing mariadb container to the newly created network bridge.
    * `docker network create --driver=bridge db-network`
    * `docker network connect db-network mariadb-diveshop`
1.  Create and run the container, connecting it to the shared network, db-network
    *`docker run --name python-app -p 5000:5000 --mount type=bind,source="${PWD}"/webapp,target=/app --net db-network munners17/python-flask`

## Login to your Flask Container
1. `docker exec -it name-of-your-flask-container /bin/sh`

* `python3`
* `4 + 3`
* `x = [1, 2, 3, 4, 5]`
* `print(x)`

## Flask Workshop

### Create the first app
Edit the file called index.py

```
from flask import Flask
app = Flask(__name__)
 
@app.route("/")
def hello():
   return "Hello World!"
 
if __name__ == "__main__":
   app.run(debug=True)
```
Open http://localhost:5000/ in your webbrowser, and “Hello World!” should appear.

### Creating URL routes
URL Routing makes URLs in your Web app easy to remember.

We will now create some URL routes:
- /destinations
- /customers/
- /members/name/

Copy the code below into index.py
```
from flask import Flask, render_template, request, redirect
app = Flask(__name__)
 
@app.route("/")
def index():
   return "Index!"
 
@app.route("/destinations")
def dest():
   return "Destinations!"
 
@app.route("/customers")
def customers():
   return "Customers"
 
@app.route("/customers/<string:name>/")
def getMember(name):
   return name
 
if __name__ == "__main__":
   app.run(debug=True)
```
Try the URLs in your browser:

- http://127.0.0.1:5000/
- http://127.0.0.1:5000/destinations
- http://127.0.0.1:5000/customers
- http://127.0.0.1:5000/customers/Jordan/
    
    
### Rendering HTML

Flask can generate HTML by referencing template files you locate in the /templates/ subdirectory. Use the render_template() function to call on the appropriate template and pass it any data you want displayed.

Create show_c.html in the templates subdirectory:
```
<html>
<head><title>INFO 257 workshop show data</title>
</head>

<h1> Hello {{customer}}</h1>

</body>
</html>
```

Edit the customers/name route in the Flask app to render the new template:

```
@app.route("/customers/<string:name>/")
def getMember(name):
   return render_template(
   'show_c.html',customer=name)
```

You can then open to see an HTML formatted page : http://127.0.0.1:5000/customers/Jackson/

### Passing Data 
The template HTML file can reference data passed by render_template() by utilizing a special syntax defined by the [Jinja2 template](https://jinja.palletsprojects.com/en/2.11.x/templates/) engine. You implemented this in the step above by passing the `name` variable to show_c.html as `customer`.

You can pass any data the python/flask app has access to, such as a local variable:

```
@app.route("/customers")
def customers():
   name_local="Keith Lucas"
   return render_template('show_c.html',customer=name_local)
```
You can then open to see an HTML formatted page with the local variable passed: http://127.0.0.1:5000/customers


#### Task: Create a Template
1.  Create a new template file (show_d.html) that prints out a destination name of your choosing when accessing this URL: `localhost:5000/destinations`


### Accessing the Database

Flask extends a library that allows for connecting to a database, SQLAlchemy: https://flask-sqlalchemy.palletsprojects.com/en/2.x/quickstart/

You will be able to execute SQL querires and receive the results programatically (similar to your DataGrip client).

First, update the python app (index.py) to connect to your Diveshop database via SQLAlchemy by adding the following lines:

```
from flask import Flask,render_template, request, redirect
from flask_sqlalchemy import SQLAlchemy


app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:mypass@mariadb-diveshop.db-network/Diveshop'
db = SQLAlchemy(app) 

@app.route("/database")
def index():
   result = db.engine.execute("SELECT DATABASE()")
   names = [row[0] for row in result]
   return names[0] 

```
`Diveshop` should be displayed when accessing : http://127.0.0.1:5000/database

### List Customers in the Database

Edit the show_c.html template to display all the customers in Diveshop. Utilize the Jinja2 template syntax referenced above to loop through the customers passed by the flask app, creating a new HTML table row for each customer.

**show_c.html**:

```
<html>
<head><title>INFO 257 workshop show data</title>
</head>

<body>
	<table class="table">
		<tr>
			<th>Customer Name</th>
			<th>City</th>
			<th>State</th>
		</tr>
			{% for ui_row in customers %}
		<tr>
			<td>{{ ui_row.Cust_Name }}</td>
			<td>{{ ui_row.City }}</td>
			<td>{{ ui_row.State }}</td>
		</tr>
			{% endfor %}
	</table>
</body>
</html>
```
**index.py:**:

Now query the database to select all customers (all rows in DIVECUST relation) in the web app and pass to rendering engine for show_c.html. Update the /customers URL:

```
@app.route("/customers")
def customers():
    result = db.engine.execute("select * from DIVECUST")
    names = []
    
    for row in result:
        name = {}
        name["Cust_Name"] = row[1]
        name["City"] = row[3]
        name["State"] = row[4]
        names.append(name)

    return render_template('show_c.html',customers=names)
   ```
   ### TASK: List Destinations
   1.  Have all the Diveshop Destinations returned when accessing http://127.0.0.1:5000/destinations
   
   ### TASK: List Destinations
   1.  Have all the Diveshop Destinations names and their travel cost display when accessing http://127.0.0.1:5000/destinations
   
   ### Forms
   Basic user input is handled by HTTP Forms.
   
   Try letting the user hit a button to choose which list of records to return from the home page.
   
   Create a new template: `index.html`:
   ```
   <html>
  <head>
    <title>INFO 257 Workshop Form</title>
    <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
  </head>

  <body>
    <br>
    
    <form method="POST" action="/customers">

      <div class="form-group">
        <input type="submit" value="List Customers" />
      </div>
    </form>
    
    <br>
    
    <form method="POST" action="/destinations">

      <div class="form-group">
        <input type="submit" value="List Destinations" />
      </div>
    </form>
    
  </body>

</html>
```

### TASK: 
1.  Update index.py to render the new index.html at the root/home location: http://127.0.0.1:5000/ . You should receive an error message declaring an unallowalbe method. To proceed:
1.  The POST method must be declared as an allowable request for the "destinatons/" and "customers/" routes. Edit existing routes in index.py that the POST in index.html is linked with:

```
@app.route("/destinations", methods=["POST"])

@app.route("/customers", methods=["POST"])
```

Click the buttons on http://127.0.0.1:5000/ to confirm they produce the correct list of data

Now try a text box to search the database. We will enter the Accomodation Type to return destinations that match that type (Expensive, Moderate, Cheap)

First add a text box form element that posts to the destinations/ URL.

**index.html**:
Add this form element after the last '</form>' tag
```
<br>
<form method="GET" action="/destinations">
	<label for="search">Find Destinations by Accomodation</label>
	<input id="search" name="search" class="form-control" type="text" /><br />
</form>
```

Now there are two form methods handled by the destionations/ flask method. Produce different logic depending on the method issued:

**index.py**:
Replace the beginning of the dest() method, everything before `names = []`, with the following:
```
if request.method == "GET":
        search = request.args.get('search')

        result = db.engine.execute("select * from DEST where Accomodations=%s",search)
   else:
        result = db.engine.execute("select * from DEST")
```
Try it out by typing an Accomodation type into the text box @ http://127.0.0.1:5000/	

Now a text box is not very friendly for matching an enumerated list of types, like Accomodation Type. Let's create a dropdown to allow the user to select from the available options.

**index.html**:
Replace the GET form elements with the following that dynamically generates the dropdown options based on the `data` variable passed:
```
<form method="GET" action="/destinations">
			<label for="search">Find Destinations by Accomodation</label>
  			<select id="accomodations" name="accomodations">
  				{% for ui_row in data %}
    			<option value="{{ui_row.Accomodations}}">{{ui_row.Accomodations}}</option>
    			{% endfor %}
    		</select>
    		<input type="submit" value="Submit" />
		</form>
```

Edit the flask route method for URL(/) to send the proper Accomodation type data to be displayed in the drop down button:
**index.py**:

```
@app.route("/")
def index():
   result = db.engine.execute("select DISTINCT(Accomodations) from DEST")
   accs = []

   for row in result:
       name = {}
       name["Accomodations"] = row[0]
       accs.append(name)

   return render_template("index.html", data=accs)
```

Right-click the web page and select "View Source" to view the web page returned back from the server to your client (browser). Verify the HTML has been updated dynamically to include all the Accomodation options:
```
<form method="GET" action="/destinations">
			<label for="search">Find Destinations by Accomodation</label>
			<!--<input id="search" name="search" class="form-control" type="text" /><br />-->
  			<select id="accomodations" name="accomodations">
  				
    			<option value="Cheap">Cheap</option>
    			
    			<option value="Moderate">Moderate</option>
    			
    			<option value="Expensive">Expensive</option>
    			
    		</select>
    		<input type="submit" value="Submit" />
		</form>
```

### TASK: Capture dropdown value
1.  Edit the destinations/ route method to capture the selected dropdown value to allow the proper Destinations to be displayed [Only requires editing one word]
1.  Verify proper destinations displayed after hitting submit button next to dropdown

   ### TODO
   1.  STYLE of HTML
   
