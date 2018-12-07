**Installing MongoDB on macOS**

1. install homebrew at brew.sh
2. `brew install mongodb`
3. Start MongoDB
    - `brew services start mongodb`                                                                         
             OR               
    - `mongod --config /usr/local/etc/mongo.conf`
4. Config is at /usr/local/etc/mongod.conf

**Installing MongoDB on Linux**

1. Visit Ubuntu setup page at mongodb.com
2. Add key: 
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
```
3. Add list file, careful on version: 
```
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
```
4. Update: `sudo apt-get update`
5. Install: `sudo apt install mongodb-org`
6. Start service: `sudo service mongod start`
7. Review config: `/etc/mongod.conf`

**Directions to restore DB into MongoDB**

To restore any of these databases to MongoDB, you'll need to uncompress them and then run this command:

```
mongorestore --drop --db DATABASE /path/to/unziped/dir
```


**Connecting**

````
$ mongo
> show dbs
> use DATABASE
> show collections
````

**Basic querying**
````
db.Test.drop()
db.Book.find().limit(1).pretty()
db.Book.find({"Title":'From the Corner of His Eye'}).count()
db.Book.find({"Title":'From the Corner of His Eye'},{Title:1, ISBN:1}).pretty()
db.Book.find({"Title":'From the Corner of His Eye'},{Title:1, ISBN:1, _id:0}).pretty()
db.Book.find({"Title":'From the Corner of His Eye', ISBN: '0553801341'},{Title:1, ISBN:1, _id:0}).pretty()
db.Book.find({"Ratings.UserId":ObjectId("525867753a93bb2198148dc0")},{Title:1,_id:0})`
````

Queries are done via find. Pass prototypical JSON documents

```
> db.Book.find({Title: 'From the Corner of His Eye'})
{
	"_id" : ObjectId("525867313a93bb2198103c40"),
	"ISBN" : "0553582747",
	"Title" : "From the Corner of His Eye",
	"Author" : "Dean Koontz",
	...
},
{
	"_id" : ObjectId("525867343a93bb21981066e7"),
	"ISBN" : "0553801341",
	"Title" : "From the Corner of His Eye",
	"Author" : "Dean R. Koontz",
	...
},
...
```

**Querying (AND)**

```
> db.Book.find({Title: 'From the Corner of His Eye', ISBN: '0553582747'})
{
	"_id" : ObjectId("525867313a93bb2198103c40"),
	"ISBN" : "0553582747",
	"Title" : "From the Corner of His Eye",
	"Author" : "Dean Koontz",
```

**Querying (sub-documents)**
```
> db.Book.find({"Ratings.UserId":ObjectId("525867733a93bb219814604e")})
{
    "_id" : ObjectId("525867313a93bb2198103c40"),
    "ISBN" : "0553582747",
    "Title" : "From the Corner of His Eye",
    "Author" : "Dean Koontz",
    "Published" : ISODate("2001-01-01T08:00:00.000Z"),
    "Publisher" : ObjectId("5258672d3a93bb21980ffff5"),
    "Ratings" : [ 
        {
            "UserId" : ObjectId("525867733a93bb219814604e"),
            "Value" : 7
        }, 
        {
            "UserId" : ObjectId("525867733a93bb21981466a0"),
            "Value" : 5
        }, 
        {
            "UserId" : ObjectId("525867733a93bb2198146914"),
            "Value" : 0
        }, 
        ...
```
**Advanced queries**

Q: How many books have been rated with 9?

`> db.Book.find({"Ratings.Value": 9}).count()
`

Q: How many books have been rated with 8 or above?

`> db.Book.find({"Ratings.Value": {$gte: 8}}).count()
`

Q: How many books have been rated above 8?

`> db.Book.find({"Ratings.Value": {$gt: 8}}).count()
`

Q: How many books have primes ratings ?

`> db.Book.find({"Ratings.Value": {$in: [1,2,3,5,7] }  }).count()
`

**Query Selectors**

Comparison
```
Name	Description
$eq     Matches values that are equal to a specified value.
$gt     Matches values that are greater than a specified value.
$gte	Matches values that are greater than or equal to a specified value.
$in     Matches any of the values specified in an array.
$lt	Matches values that are less than a specified value.
$lte	Matches values that are less than or equal to a specified value.
$ne	Matches all values that are not equal to a specified value.
$nin	Matches none of the values specified in an array.
```
Logical
```
Name	Description
$and	Joins query clauses with a logical AND returns all documents that match the conditions of both clauses.
$not	Inverts the effect of a query expression and returns documents that do not match the query expression.
$nor	Joins query clauses with a logical NOR returns all documents that fail to match both clauses.
$or	Joins query clauses with a logical OR returns all documents that match the conditions of either clause.
```
https://docs.mongodb.com/manual/reference/operator/query/


**Projections**

```
> db.Book.find({...},{ISBN:1,Title:1})

{
    "_id" : ObjectId("525867313a93bb2198103c40"),
    "ISBN" : "0553582747",
    "Title" : "From the Corner of His Eye"
}

{
    "_id" : ObjectId("525867323a93bb2198104f24"),
    "ISBN" : "3404136012",
    "Title" : "Wintermond. Unheimlicher Roman."
}

{
    "_id" : ObjectId("525867323a93bb2198104f25"),
    "ISBN" : "3453131169",
    "Title" : "Fl�?¼stern in der Nacht."
}
```

**Exact subdocument matches**

That's not the AND we wanted

```
> db.Book.find({"Ratings.Value":9,"Ratings.UserId":600)})

{...
    "Ratings" : [ 
        {
            "UserId" : 700,
            "Value" : 5
        }, 
        {
            "UserId" : 200,
            "Value" : 9
        }, 
        {
            "UserId" : 600,
            "Value" : 0
        }, ...
},
{...
    "Ratings" : [ 
        {
            "UserId" : 600,
            "Value" : 9
        }, ... 
},

//more results
```

We want $elemMatch

```
> db.Book.find({Ratings: {$elemMatch: {UserId:600), Value: 9} }})

{...
    "Ratings" : [ 
        {
            "UserId" : 600,
            "Value" : 9
        }, ... 
},

//more results
```

**Sorting**


`> db.Book.find().sort( { Published: -1} )
`
```
> db.Book.find().sort( {Title:1, Published: -1} )

{
    "_id" : ObjectId("525867433a93bb219811638a"),
    "Title" : "!%@ (A Nutshell handbook)",
    "Published" : ISODate("1994-01-01T08:00:00.000Z")
}

/* 2 */
{
    "_id" : ObjectId("525867563a93bb2198129ecf"),
    "Title" : "!%@ (A Nutshell handbook)",
    "Published" : ISODate("1993-01-01T08:00:00.000Z")
}

/* 3 */
{
    "_id" : ObjectId("5258674c3a93bb219811f52c"),
    "Title" : "!Arriba! Comunicacion y cultura",
    "Published" : ISODate("1996-01-01T08:00:00.000Z")
}
```

**Inserts**

If we don't specify key: _id, MongoDB will generate it.
```
> db.Book.insert({Title: 'From the Corner of My Eye', ...'})

> db.Book.find()...

{
    "_id" : ObjectId("5c0437172d5576eb08ded262"),
    "Title" : "From the Corner of My Eye",
    "ISBN" : "0440234743",
    "Author" : "Steve Jobs"
    //...
}
```

**Whole document Update**

```
> db.Book.update(
        {_id : ObjectId("5c0437172d5576eb08ded262") },                                            # First argument (required) is the WHERE clause
        {Title: 'From the Corner of My Eye 2nd Edition', ISBN: '0000000000',Author: 'Mr.NoName'}, # Next argument (required) is the new document
        {upsert: true, multi: true} )                                                             # Additional options may not be specified

> db.Book.find()...

{
    "_id" : ObjectId("5c0437172d5576eb08ded262"),
    "Title" : "From the Corner of My Eye 2nd Edition",
    "ISBN" : "0000000000",
    "Author" : "Mr.NoName"
}

```

**Deleting documents**

```
> db.Book.deleteOne( {"_id" : ObjectId("5c0437172d5576eb08ded262")} )

> db.Book.deleteMany( {"Title" : "Some title"} )
```

**Atomic updates**



```
> var book = db.getCollection('BookReads').findOne({'ISBN':'94724773501'})
> book.ReadCount += 1
> db.getCollection('BookReads').update({'_id':book._id},book)
> db.getCollection('BookReads').find({'ISBN':'94724773501'})


{
    "_id" : ObjectId("5c0506042d5576eb08ded263"),
    "ISBN" : "94724773501",
    "ReadCount" : 1.0
}
```

Better way for update is to use operators.

```
> db.getCollection('BookReads').find({'ISBN':'94724773501'})
> db.getCollection('BookReads').update({'_id':book._id},{$inc: {ReadCount: 1}})
> db.getCollection('BookReads').find({'ISBN':'94724773501'})


{
    "_id" : ObjectId("5c0506042d5576eb08ded263"),
    "ISBN" : "94724773501",
    "ReadCount" : 2.0
}

```


Field
```
Name	        Description
$currentDate	Sets the value of a field to current date, either as a Date or a Timestamp.
$inc	        Increments the value of the field by the specified amount.
$min	        Only updates the field if the specified value is less than the existing field value.
$max	        Only updates the field if the specified value is greater than the existing field value.
$mul	        Multiplies the value of the field by the specified amount.
$rename	        Renames a field.
$set	        Sets the value of a field in a document.
$setOnInsert	Sets the value of a field if an update results in an insert of a document. Has no effect on update operations that modify existing documents.
$unset	        Removes the specified field from a document.
```


Array

```
Operators

Name	        Description
$	        Acts as a placeholder to update the first element that matches the query condition.
$[]	        Acts as a placeholder to update all elements in an array for the documents that match the query condition.
$[<identifier>]	Acts as a placeholder to update all elements that match the arrayFilters condition for the documents that match the query condition.
$addToSet	Adds elements to an array only if they do not already exist in the set.
$pop	        Removes the first or last item of an array.
$pull	        Removes all array elements that match a specified query.
$push	        Adds an item to an array.
$pullAll        Removes all matching values from an array.
```

https://docs.mongodb.com/manual/reference/operator/update/
```
> db.Test.insert( { Title: 'A popular book', ViewCount: 0} )
> db.Test.find()

{
    "_id" : ObjectId("5c050ca62d5576eb08ded264"),
    "Title" : "A popular book",
    "ViewCount" : 0.0
}

> db.Test.update( {_id:ObjectId("5c050ca62d5576eb08ded264")}, {$inc: {ViewCount: 1}})
> db.Test.update( {_id:ObjectId("5c050ca62d5576eb08ded264")}, {$inc: {ViewCount: 1}})
> db.Test.update( {_id:ObjectId("5c050ca62d5576eb08ded264")}, {$inc: {ViewCount: 1}})

{
    "_id" : ObjectId("5c050ca62d5576eb08ded264"),
    "Title" : "A popular book",
    "ViewCount" : 3.0
}

```

#Introduction to PyMongo

_pymongo_ is the core package to access MongoDB

Features include
- Connect to database, replicate set, or shard
- Query and generally perform CRUD
- Other admin operations
- Connection pooling

https://github.com/mongodb/mongo-python-driver

Some CRUD operations

```
import pymongo

conn_str = 'mongodb://localhost:27017'
client = pymongo.MongoClient(conn_str)

db = client.the_small_bookstore

# now we can operate on the db via collections
print('There are {} books'.format(db.books.count() ))
print('First book: {}'.format(db.books.find_one() ))
print('Book by ISBN: {}'.format(db.books.find_one({'ISBN': '0399135782'}) ))

res = db.books.insert_one({'title': 'New book','ISBN': '1234567890'})

{
    "_id" : ObjectId("5c0518e2bd09bd34373e8528"),
    "title" : "New book",
    "ISBN" : "1234567890"
}
 
```

**Connection string examples**

Connect to the server mongo_server on default port within a virtual private network or in the same data center zone or cloud hosting like a Digital Ocean

`conn_str = 'mongodb://mongo_server'
`

Connect to mongo_server on an alternate port

`conn_str = 'mongodb://mongo_server:2000' 
`

Use authentication when connecting

`conn_str = 'mongodb://jeff:supersecure@mongo_server:2000' 
`

Connect to a replicate set

`conn_str = 'mongodb://mongo_server:2000, mongo_server:2001, mongo_server2:2002/?replicaSet=prod' 
`

**Atomic updates from Python (using the in_place operators)**
```
import pymongo
conn_str = 'mongodb://localhost:27017'
client = pymongo.MongoClient(conn_str)
db = client.the_small_bookstore

res = db.books.insert_one({'title': "New Book", 'isbn': "1234567890"})
db.books.update({'isbn': "1234567890"}, {'$addToSet': {'favorited_by': 1001}})
db.books.update({'isbn': "1234567890"}, {'$addToSet': {'favorited_by': 1002}})
db.books.update({'isbn': "1234567890"}, {'$addToSet': {'favorited_by': 1002}})

{'_id': ObjectId('5c0646b4bd09bd3d6f1a371b'), 'title': 'New Book', 'isbn': '1234567890', 'favorited_by': [1001, 1002]}

```

**EXAMPLE: Atomic vs Whole document update from Python**

```
import pymongo

conn_str = 'mongodb://localhost:27017'
client = pymongo.MongoClient(conn_str)

db = client.the_small_bookstore

if db.books.count() == 0:
    print("Inserting data")
    # insert some data...
    r = db.books.insert_one({'title': 'The first book', 'isbn': '73738947384'})
    print(r, type(r))
    r = db.books.insert_one({'title': 'The second book', 'isbn': '73738947385'})
    print(r.inserted_id)
else:
    print("Books already inserted, skipping")


 ''' To pull whole document back and update'''

# book = db.books.find_one({'isbn': '73738947384'})
# # print(type(book),book)
# # book['favorited_by'] = []
# book['favorited_by'].append(42)
# db.books.update({'_id':book.get('_id')},book)
# book = db.books.find_one({'isbn': '73738947384'})
# print(book)

'''An atomic, in place update'''

db.books.update({'isbn': '73738947385'}, {'$addToSet': {'favorited_by': 101} } ) # addToSet Mongo operator in Python with quotes
book = db.books.find_one({'isbn': '73738947385'})
print(book)


{'_id': ObjectId('5c0518e3bd09bd34373e8529'), 'title': 'The second book', 'isbn': '73738947385', 'favorited_by': [101]}
```

Mapping from MongoDB api to PyMongo documentation

http://api.mongodb.com/python/current/api/pymongo/collection.html

_Note: If we want to write an app, PyMongo could be our data access layer, the low level way to talk to MongoDB._

# Modeling and document design
![alt text](src/pic28.png)


![alt text](src/pic29.png)

**To embed or not to embed (normalized) data?**

1. is the embedded data wanted **80% of the time**?
2. How often do you want the embedded
   data **without the containing document**? (if often -> normalize)
3. Is the embedded data **a bounded set**? 
4. Is that bound **small**?
5. **How varied** are your queries? (if much -> normalize)
6. Is this an **integration DB** or an **application DB**? 


Do we have an integration database?
![alt text](src/pic30.png)
Especially in large enterprises, we'll see that they use databases almost as a means of inter-application communication,
so maybe we have this huge relational database that lives in the center with many, many constraints, many store procedures, 
lots and lots of structures and rules. Because we have a bunch of different applications and they all need to access this data,
This is a decent, a good role for relational databases, but relational databases are a good guarding against this kind of use case,
they have a fixed schema, they have lots of constraints and relationships and they are very good at enforcing and kicking it back to the app
and go no, you got it wrong, you messed up the data.
So they can be like this strong rock in the middle.
The problem with rocks is they're not very adaptable, they can't be massaged into new and interesting things; a rock is a rock, and it's extremely hard to change.
It's also not a great use case for document databases with their flexibility in schema design, their less enforcement at the database level and more enforcement inside the app.

This is an integration database, and it's generally not a good use case for document databases, if we're still using that this sort of style of document databases, 
it means our queries will be more varied and we probably need to model in a more relational style,less embedded style, just as a rule of thumb.


![alt text](src/pic31.png)

Each one of these little apps is much simpler, it can have its own DB with its own focused query patterns.
When we have an application DB like this, we are more likely to have slightly
more embedded objects because the query patterns are going to be simpler and more focused and more constraints.

**Document patterns**

MongoDB Applied Design Patterns  
https://amzn.to/2qx47oL

Episode#109: MongoDB Applied Design Patterns
https://talkpython.fm/109


**Mapping classes to MongoDB with the ODM MongoEngine**

![alt text](src/pic32.png)

Mongoengine is a Document-Object Mapper (think ORM, but for document databases) for working with MongoDB from Python.
It uses a simple declarative API, similar to the Django ORM. 
Documentation available at 
http://docs.mongoengine.org - there is currently a tutorial, a user guide and API reference.

Installing Mongoengine
```
(venv) MacBook-Pro-xxx:mongoDB_basic_syntax xxx$ pip list
Package    Version
---------- -------
pip        10.0.1 
pymongo    3.7.2  
setuptools 39.1.0 
You are using pip version 10.0.1, however version 18.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
(venv) MacBook-Pro-xxx:mongoDB_basic_syntax xxx$ pip install mongoengine
Collecting mongoengine
```
Registering connections in mongoengine

In order to use MongoEngine we need to do some configuration of MongoEngine.
Create separate folder in your app (nosql in my case) with connection module called **mongo_setup.py**.
In this module we need to set up the connections with aliases to the application classes. 


```
import mongoengine

def global_init():
    pass
```

In `global_init()` function we need to register the connection. We are not going to open the connection, it doesn't talk to the 
database, but it basically says look if you have a class that maps to a particular type or named part of our application
use this database connection to do the backend work.

```
import mongoengine

def global_init():
    mongoengine.register_connection(alias='core', name='demo_dealership')
    mongoengine.register_connection(alias='analytics', name='demo_dealership_visits')
```

So in our app we can say this class belongs to the core database and this one over here belong to analytics database. 
In production we will pass the extra information we need to use for real server on another port with authentication.

In **service_app.py** file we need to `import mongo_setup` module and define `config_mongo()` method which will run
`mongo_setup.global_init()` 
function inside.

```
import nosql.mongo_setup as mongo_setup

def main():
    print_header()
    config_mongo()
    user_loop()


def print_header():
    print('----------------------------------------------')
    print('|                                             |')
    print('|           SERVICE CENTRAL v.02              |')
    print('|               demo edition                  |')
    print('|                                             |')
    print('----------------------------------------------')
    print()

def config_mongo():
    mongo_setup.global_init()


def user_loop():
    while True:
        ...
     
```

Now we can start defining our classes that we are going to map to the db.
First we need to decide how we are going to store the data and what the data is.

So we are going to have a car, a car is going to have an engine with lots of details about the engine, like its horsepower
and so on. A car is going to have a service history and each entry in the services history is going to be some additional
information, like what was the work performed, how much did it cost, when was it done, etc.

There is going to be an owner who can own multiple cars and a car can be owned by multiple people, so there is a many to
many relationship between owners and cars. The owners have personal information like their address and stuff like that.

We are going to create a file Car in the nosql folder with definition of class for Car object.

We want all the classes, which we want to map to the db, derive from mongoengine.Document. This allows us to load, save 
and query documents. It also provides a field called id, which maps to undescore id in the db. We are going to give it a
couple of pieces of information like what model is it, what make, etc. So we define the properties of document as a descriptor,
so it's a mongoengine. We need also to define the meta dictionary and dictionary is going to say the db alias we want to use is
'core', then we can also control the name of the collections.

nosql/car.py
```
import mongoengine

class Car(mongoengine.Document):
#    id # --> _id = ObjectId()...
    model = mongoengine.StringField()
    make = mongoengine.StringField()
    year = mongoengine.IntField()
    mileage = mongoengine.FloatField()
    vi_number = mongoengine.StringField()

    meta = {
        'db_alias':'core',
        'collection': 'cars',
    }
```

In main application service_app.py we define add_car() function, where we initialize the car object and save it to DB.

service_app.py
```
import nosql.mongo_setup as mongo_setup

from mongoengine.service_central_starter.nosql.car import Car


def main():
    print_header()
    config_mongo()
    user_loop()

...

def add_car():
#    print("TODO: add_car")
    model = input('What is the model?')
    make = input('What is the make?')
    year = int(input('Year built?'))
    mileage = float(input('Mileage?'))
    vin = input('VIN? ')

    car = Car()
    car.model = model
    car.make = make
    car.year = year
    car.mileage = mileage
    car.vi_number = vin

    car.save()   # in order to insert it to db in active record style, where we work with a single document

...

```

We can update the car script with the required fields for the car class.
```
import uuid
import mongoengine


class Car(mongoengine.Document):
#    id # --> _id = ObjectId()...
    model = mongoengine.StringField(required=True)
    make = mongoengine.StringField(required=True)
    year = mongoengine.IntField(required=True)
    mileage = mongoengine.FloatField(default=0.0)
    vi_number = mongoengine.StringField(default=lambda: str(uuid.uuid4()).replace('-',''))

    meta = {
        'db_alias':'core',
        'collection': 'cars',
    }
```

and delete the auto generated variables from add_car() function in main application.


```
def add_car():
    model = input('What is the model?')
    make = input('What is the make?')
    year = int(input('Year built?'))
    # mileage = float(input('Mileage?'))
    # vin = input('VIN? ')

    car = Car()
    car.model = model
    car.make = make
    car.year = year
    # car.mileage = mileage
    # car.vi_number = vin

    car.save()   # in order to insert it to db in active record style, where we work with a single document
```

The next thing we want to look at is the engine and the embedded elements. The engine will equal to a subclass, a class that
represents engines. We define a class Engine in separata file, which will derive from mongoengine as a embedded subdocument.
We will not query and save them independently, we can only work with them through their parent document. 

nosql/engine.py
```
import uuid
import mongoengine

class Engine(mongoengine.EmbeddedDocument):
    horsepower = mongoengine.IntField(required=True)
    liters = mongoengine.FloatField(required=True)
    mpg = mongoengine.FloatField(required=True)
    serial_number = mongoengine.StringField(default=lambda: str(uuid.uuid4()))
```

nosql/car.py

```
import uuid
import mongoengine

from nosql.engine import Engine


class Car(mongoengine.Document):
#    id # --> _id = ObjectId()...
    model = mongoengine.StringField(required=True)
    make = mongoengine.StringField(required=True)
    year = mongoengine.IntField(required=True)
    mileage = mongoengine.FloatField(default=0.0)
    vi_number = mongoengine.StringField(default=lambda: str(uuid.uuid4()).replace('-',''))

    engine = mongoengine.EmbeddedDocumentField(Engine,required=True)

    meta = {
        'db_alias':'core',
        'collection': 'cars',
    }
```

service_app.py

