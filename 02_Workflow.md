
##################
## MONGO BASICS ##
##################

* INSTALLATION
*****************

 Installation is srightforward. Just extract the binaries or install the desired package onto the computer.
	
	>\bin\mongod.exe --dbpath "d:\set up\mongodb\data"			# To init the data base

	>\bin\mongod.exe --auth --dbpath "d:\set up\mongodb\data"	# To init the data base with auth mode enabled

	>\bin\mongod --auth --port 27017 --dbpath /data/db1		# Run the database with Auth enabled, current port and database path 

	>\bin\mongo.exe 											# To access to the data base (client)

	>Data base will be deployed at "mongodb://localhost:27017"


* DATA BASES
***************

	>show dbs						 # Show all created data bases in mongodb. Data base must have some data already to show up.

	>use mydb						 #used a data base or create a new one

	>db						  		 # Show the current database being used

	>db.dropDatabase()				 # Remove the current database


* USERS
*********

	>db.auth( <username>, <password> )															# To authenticate into the data base

	>db.createUser({user: "accountUser",pwd: "password",roles: [ "readWrite", "dbAdmin" ]})		# Create a user with roles
	>db.createUser( { user: "reportsUser", pwd: "password", roles: [ ] })						# Creates a user withot roles

	>db.updateUser( "appClient01",{customData : { employeeId : "0x3039" }roles : [{ role : "read", db : "assets"  } ]} ) # Updates user data

	>db.changeUserPassword("accountUser", "SOh3TbYhx8ypJPxmt1oOfL")								# Changes used password

	>db.dropUser("reportUser1")																# Remove the user from database


* COLLECTIONS
***************

	>show collections								 # Show all collections created in the current db 
	
	>db.movie.insert({"name":"tutorials point"})	 # Creates a new collection called movie and insert a new item (document).

	>db.createCollection("films")					 #Creates a new collection. Not neccesary since it will be created autmatically when insert the first item.

* DOCUMENTS 
*************

	 - INSERT

		>db.movie.insert({"name":"tutorials point"})							# Creates a new collection called movie and insert a new item (document).
	
		>db.movie.insert([{"name":"Jurassic Park"},{"name":"Spiderman"}])		# Insert Multiple elements, like an array

	- UPDATE

		>db.mycol.update({'title':'MongoDB Overview'},{$set:{'title':'New MongoDB Tutorial'}}					# Update the element that match with the find #1 and set the new params

		>db.mycol.update({'title':'MongoDB Overview'}, {$set:{'title':'New MongoDB Tutorial'}},{multi:true})	# Update multiple elements. By default only the first match is updated

		>db.mycol.save({_id:ObjectId(),NEW_DATA})																# Replace the entire object with the ID

	- REMOVE

		>db.mycol.remove({'title':'MongoDB Overview'})					# Remove the doucment with the criteria													

	- QUERY

		>db.movie.find()										# It returns all the iem in the collection "movie". 

		>db.movie.find().pretty()								# It returns all the iem in the collection "movie". Uee pretty to get a formatted input.

		>db.movie.find("name":"tutorials point"}).pretty()		# Find an item. For more operations see http://www.tutorialspoint.com/mongodb/mongodb_query_document.htm
	
			# quality 	{<key>:<value>} 	db.mycol.find({"by":"tutorials point"}).pretty() 	where by = 'tutorials point'
			# Less Than 	{<key>:{$lt:<value>}} 	db.mycol.find({"likes":{$lt:50}}).pretty() 	where likes < 50
			# Less Than Equals 	{<key>:{$lte:<value>}} 	db.mycol.find({"likes":{$lte:50}}).pretty() 	where likes <= 50
			# Greater Than 	{<key>:{$gt:<value>}} 	db.mycol.find({"likes":{$gt:50}}).pretty() 	where likes > 50
			# Greater Than Equals 	{<key>:{$gte:<value>}} 	db.mycol.find({"likes":{$gte:50}}).pretty() 	where likes >= 50
			# Not Equals 	{<key>:{$ne:<value>}} 	db.mycol.find({"likes":{$ne:50}}).pretty() 	where likes != 50

			# AND: db.mycol.find({"by":"tutorials point","title": "MongoDB Overview"}).pretty()
			# OR:  db.mycol.find({$or:[{key1: value1}, {key2:value2}]}).pretty()
			# AND OR TOGETHER: db.mycol.find({"likes": {$gt:10}, $or: [{"by": "tutorials point"}, {"title": "MongoDB Overview"}]}).pretty()


		>db.mycol.find({},{"title":1,_id:0}).sort({"title":-1})		# Sort the find values by title key

* BACKUP 
**********

	$.\bin\mongodump.exe --db test													# This will create a dump of the database in json format creating also a "\dump" folder. The database is given in the paramater.
	$.\bin\mongodump.exe --db test --out - | gzip > dump_`date "+%Y-%m-%d"`.gz		# Zip the dump files

	$.\bin\mongorestore -d test ".\dump\test"										# 
	
