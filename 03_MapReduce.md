# Map-Reduce
##############

Accordingly to Wikipedia, MapReduce is a programing model of processing and generating large data sets with a Parallel, Distributed algorithm on a 
cluster. The input data is processed in pararel between the different workers that will generate the data requiered ofr each step of Mapreduce.

>- "Map" step: Each worker node applies the "map()" function to the local data, and writes the output to a temporary storage. A master node ensures that only one copy of redundant input data is processed.
  
>- "Shuffle" step: Worker nodes redistribute data based on the output keys (produced by the "map()" function), such that all data belonging to one key is located on the same worker node.

>- "Reduce" step: Worker nodes now process each group of output data, per key, in parallel.


MongoDB also has the functionality to apply a MapRecuce function to the data. Map Reduce is used to compute and store the data in parallel between nodes (Master and slaves). 
Because the amount of information and the number of sources some data requires this paradigm has been developed to optimize this process. 

- First we need the data that is going to be stored into the Database. The data sets will contain a number of fields (data-scheme) and tuples with the data.

- The MapReduce process is divided into two operations: Map and Reduce.
	
	Map: is a function that returns a <key, value> pair for each tuple of data stored. This will be chosen depending on the requeriments of the data and the needs.
		> Map function is originally created to change the domain of the data into another, more convenient to the main focus and purpose of the data.
		> Because the key could not be unique, the Map function will group all the results into a list of values for each key.
		> the final result is a list of tuples with a <key,value> pair.

	Since the infrastrucutre is originall created to be a networks node base with multiple "workers" and one Master that manage all the process. The system will have
	a list will all the Maps created for each workers. For this reason the Reduce function takes place.

	(Each Key is located in the same worker node. This step is called (shuffle steps))

	Reduce: is another function, that takes each of the <key, list<values>> and generate another value for eeach key. The list of values for each key will depend on the 
	distributed systems and the number of nodes. 

- THe idea behind this algorithm isthat  we could have different mappings operatons independent from each other. So we could access to the information that we need in
short period of time.


-the Logical View of Map convert one data pair domain into another data pair domain.  Map(k1,v1) ? list(k2,v2)  -> See it returns a list of key par values

************
*EXAMPLE: 
**************
	
 - *INPUTS*

 - We have a document that have words in it. e.g.
		Line 234 "the cat is on the table"
		Line 235 "because the bed is broken"

		# In this case the document are ordered using the following domain.
		Domain -> Key: number of line
				  Value: phrase with worids

 - *MAP*

 - For each worker the input will be distributed in parallel to be processed.

 - The pseudocode of the Map function, and for this example, is the following:

		def map(key, Value)
			For each word in input
				# where 1 is the word counter to be used later by the Reduce function... is the simples use case
				emit (word, 1)						

 - This function basically will transform the input domain <key,value> (line, phrase) into another a different domain <word, count>

 - Map function will generates the following output in the first iteration:

		[(the,1), (cat,1), (is,1), (on,1), (the,1), (table,1), (because,1), (the,1), (bed,1), (is,1), (broken,1)]

 - Each <key, pair> will be grouped into a list.
		[(the, [1,1]),
		(cat, [1]),
		(is, [1,1]),
		(on, [1]),
		...
		(broken, [1]
		
		)]
		
 - *REDUCE*

 - Finally Reduce function will take the outputs from all the workers and will execute the Reduce function. This function will generate a <key, value> pair element.

		def reduce (key, list <values>)
			set count = 0
			for each item in list <values>
				count += item

			emit (key, count)

 - The output from this function will be:
		( (the,2), (cat, 1), (is,2), (on, 1), .. , (broken, 1))
		


	
