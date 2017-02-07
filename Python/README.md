# MongoDB for Python

This document has been written to familiarize with this database and some of the basic operations:

1. Create new data base
2. Create new collection
3. Add/Update/Remove item from collection
4. Querys (optimized for Non-SQL)

##1.References

See for further documentation
http://www.tutorialspoint.com/mongodb/mongodb_create_database.htm

Good Tutorial about MongoDB using python 
http://api.mongodb.com/python/current/tutorial.html

##2. Implementation

###2.1 Modules

First you need to import some libraries to be able to connect to MongoDB

	import pymongo
	from pymongo import MongoClient as mongodb
	import datetime

###2.2 Connection

###2.2.1 Connect to an exisisting data base.

	>In my case I installed the server in the current address from local: #mongodb://localhost:27017
	
	client = mongodb("mongodb://localhost:27017")

###2.2.2 Connection Get all the databases in the current Client
	
	dbs = client.database_names()
	print (dbs)
	
	[u'local', u'test', u'test2']

	> Using the following commands you will get an error if you have configured the database like me 
	db = client.get_default_database() # Error with no database defined
	print (db)

###2.2.3 Get the current data base.

	db = client.test
	db = client["test"]
	
###2.3 Collections
	
####2.3.1 Create new collection
	
	articles = db.Articles
	post = {"name":"The Title", "description":"This is something", "date":datetime.datetime.utcnow()}
	document = articles.insert_one(post)
	id_document = document.inserted_id

####2.3.2 List all the collection in the data base

	print (db.collection_names(include_system_collections=False))

	post = {"name":"The Title2", "description":"This is something2", "date":datetime.datetime.utcnow()}
	document = articles.insert_one(post)

	print (articles.find_one({"name":"The Title2"}))
	print (articles.find_one({"name":"error"})) # No result found

	print (id_document)
	articles.find_one({"_id": id_document})


####2.3.3  Insert multiple documents into a collection

	new_articles = [{"author": "Mike",
				  "text": "Another post! But not the first one",
				  "tags": ["bulk", "insert"],
				  "date": datetime.datetime(2009, 11, 12, 11, 14)},
				 {"author": "Eliot",
				  "title": "MongoDB is fun, but not the first one",
				  "text": "and pretty easy too!",
				  "date": datetime.datetime(2009, 11, 10, 10, 45)}]
	result = articles.insert_many(new_articles)
	print (result)

	print (articles.find_one({"author":"Eliot"}))

###2.4 Queries

####2.4.1 Query in where the response will result multiple number

	print (articles.find({"author": "Mike"}).count())
	print (articles.find().count())

####2.4.2 The result obtaind from the query is yield into a generator so it must be obtained this way.
	
	for article in articles.find():
		print (article)

####2.4.3 The result obtaind from the query is yield into a generator so it must be obtained this way.
	
	for article in articles.find({"author": "Mike"}):
		print (article)

###2.5 Updates

####2.5.1 Single Update

	print (articles.update_one({"author":"Eliot"}, {"$set": {"text": "Another post! But not the first one and not the second one"}}))

	> The result obtaind from the query is yield into a generator so it must be obtained this way.
	
	for article in articles.find({"author": "Eliot"}):
		print (article)


####2.5.2 Multiple Update 

For multiple updates is also possible to specify if the docuemnts don't exist then insert them (upsert = true)

	update_many(filter, update, upsert=False, bypass_document_validation=False) -> HEARDER of the function

	print (articles.update_many({"author":"Mike"}, {"$set": {"text": "Multiple Update"}}))

The result obtaind from the query is yield into a generator so it must be obtained this way.

	for article in articles.find({"author": "Mike"}):
		print (article)

###2.6 Delete

####2.6.1 Single Delete 

	result = articles.delete_one({"author": "Javier"})
	print (result.deleted_count)
	print (articles.find({"author": "Javier"}).count())
	for article in articles.find({"author": "Javier"}):
		print (article)

####2.6.2 Delete multiple documents

	result = articles.delete_many({"author": "Javier"})
	print (result.deleted_count)
	print (articles.find({"author": "Javier"}).count())
	for article in articles.find({"author": "Javier"}):
		print (article)
	
###2.7 Indexing

###2.7.1 Create Indexes

In order to create the indexes first the index must be created.
	
	result = articles.create_index([('article_id', pymongo.ASCENDING)], unique=True)
	print (result)

	print (list(articles.index_information()))

MongoDB also use an id to indentify each document, but these indexes prevent to the database to not insert documents that are already created.
In order to inserte a new article we must provide the index create with the .

	new_articles = [{"user_id": 101,
				  "author": "Javier",
				  "text": "Another post! But not the first one",
				  "tags": ["bulk", "insert"],
				  "date": datetime.datetime(2009, 11, 12, 11, 14)},
				 {"user_id": 102,
				  "author": "Pedro",
				  "title": "The last of US",
				  "text": "and pretty easy too!",
				  "date": datetime.datetime(2009, 11, 10, 10, 45)}]
	result = articles.insert_many(new_articles)
	print (result)
	print (articles.find().count())

###2.7.3 Remove indexes

Remove the index specified

	articles.dropIndex( { "article_id": 1 } )

Remove all the idnexes

	articles.dropIndexes()
	
###2.8 Other

Since this is intended to be a very basic tutorial, there are so many thing that will be covered in another document:

- Map Reduce
- Authorization
- Users