# Installation

## Install On Windows

- To install the MongoDB on windows, first download the latest release of MongoDB from http://www.mongodb.org/downloads Make sure you get correct version of MongoDB depending upon your windows version. To get your windows version open command prompt and execute following command.

		C:\>wmic os get osarchitecture
		OSArchitecture
		64-bit
		C:\>

 > 32-bit versions of MongoDB only support databases smaller than 2GB and suitable only for testing and evaluation purposes.

- Now extract your downloaded file to c:\ drive or any other location. Make sure name of the extracted folder is _mongodb-win32-i386-[version]_ or _mongodb-win32-x86_64-[version]_. 

- Now open command prompt and run the following command

		C:\>move mongodb-win64-* mongodb
		1 dir(s) moved.
		C:\>

 In case you have extracted the mongodb at different location, then go to that path by using command cd FOOLDER/DIR and now run the above given process.

 MongoDB requires a *data folder* to store its files. The default location for the MongoDB data directory is _c:\data\db_. So you need to create this folder using the Command Prompt. Execute the following command sequence.

		C:\>md data
		C:\>md data\db

 If you have installed the MongoDB at different location, then you need to specify any alternate path for _\data\db_ by setting the path _dbpath_ in mongod.exe. In this case, I will install mongodb in different location.

- In command prompt navigate to the bin directory present into the mongodb installation folder. Suppose my installation folder is _D:\setup\mongodb_
 
		C:\Users\XYZ>d:
		D:\>cd "setup"
		D:\setup>cd mongodb
		D:\setup\mongodb>cd bin
		D:\setup\mongodb\bin>mongod.exe --dbpath "d:\setup\mongodb\data" 

 This will show waiting for connections message on the console output indicates that the mongod.exe process is running successfully.

- Now to run the mongodb you need to open another command prompt and issue the following command.
	 
		D:\setup\mongodb\bin>mongo.exe
		MongoDB shell version: 2.4.6
		connecting to: test
		>db.test.save( { a: 1 } )
		>db.test.find()
		{ "_id" : ObjectId(5879b0f65a56a454), "a" : 1 }
		>

- This will show that mongodb is installed and run successfully. Next time when you run mongodb you need to issue only commands.

 - Start Server
 
		D:\setup\mongodb\bin>mongod.exe --dbpath "d:\setup\mongodb\data" 
		
 - Connection Client
 
		D:\setup\mongodb\bin>mongo.exe
	
- Connection string URL 

	> This is used to connect through mongoose or other library. e.j. connect('mongodb://localhost:27017'); 
	
 To describe a connection to a replica set named test, with the following mongod hosts:

		db1.example.net on port 27017 and
		db2.example.net on port 2500.

 You would use a connection string that resembles the following:

		mongodb://db1.example.net,db2.example.net:2500/?replicaSet=test
		mongodb://localhost:27017/?replicaSet=test or mongodb://localhost:27017

##References 

http://www.tutorialspoint.com/mongodb/mongodb_environment.htm

