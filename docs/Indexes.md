# Indexes

db.coll.getIndexes()

<br>

- Deleting an index that is the only index supporting a query will affect the performance of that query. You should hide the index before deleting it. This way, you'll be able to assess the impact of removing the index on query performance. MongoDB does not use hidden indexes in queries but continues to update their keys. This allows you to assess if removing the index affects the performance and unhide the index if needed. Unhiding an index is faster than recreating it.

<br>

INDEXES
- Without indexes, MongoDB must perform a collection scan, i.e. scan every document in a collection.
	- If an appropriate index exists for a query, MongoDB can use the index to limit the number of documents it must inspect.
- Indexes are special data structures that store a small portion of the collection's data set in an easy to traverse form
- Each index has a name. If not specified, the name is created automatically using the index key
- Use .explain("executionStats") to analyze query performance

```js
db.col.createIndex({a: 1})
db.col.createIndex({a:1, b:1})  //order is important, es important quin index es posa primer
db.col.createIndex({a:1, b:-1}) //MALAMENT, no es pot posar un positiu i un negatiu

db.col.dropIndex({a:1})

db.col.getIndexes()  //to list the indexes
```


<br>


### COLLECTION SCAN
- when every document in the collection must be checked in order to determine the result set of a query
- per saber si ha passat mirem el .explain() -> queryPlanner -> winningPlan -> COLLSCAN
- per saber si hem utilitzat indexes mirem el .explain() -> executionStats -> totalKeysExamined


<br>


### SINGLE FIELD INDEXES
```js
db.col.createIndex({a: 1})
```

- Index on a single field
- We can index fields inside embedded documents too
- We can index embedded documents too
- If we query on multiple fields and only one of them is indexed, mongoDB will filter using the index first


<br>


### COMPOUND INDEXES
- An index that contains references to multiple fields within a document
- The recommended order of indexed fields in a compound index is Equality, Sort, and Range. Optimized queries use the first field in the index, Equality, to determine which documents match the query. The second field in the index, Sort, is used to determine the order of the documents. The third field, Range, is used to determine which documents to include in the result set.

```js
db.col.createIndex({a:1, b:1})
```

- Ex: Index { "item": 1, "location": 1, "stock": 1 }
	- The index has the following index prefixes: { item: 1 } and { item: 1, location: 1 }
	- For a compound index, MongoDB can use the index to support queries on the index prefixes.
	- As such, MongoDB can use the index for queries on the following fields:
		#item field
		#item field and the location field
		#item field and location field and stock field.


<br>


### MULTIKEY INDEXES
- MongoDB automatically creates a multikey index if any indexed field is an array. 
- Creates an index key for each element in the array. This allows MongoDB to search for the index key of each element in the array rather than scan the entire array, which results in dramatic performance gains in your queries.
- Careful when creating multikey indexes. We want to make sure that our arrays don't grow too large.
- If we create a multikey compound index, we have to make sure that we only have one field that is an array.
	- if we index two array fields in the same compound index, we would be generating a big amount of index keys.
	- Ex: index on “name” and “emails” with a collection containing: 
	```js
	{ 
		"name": "Beatrice McBride", 
		"age": 26, 
		"emails": [ "puovvid@wamaw.kp", "todujufo@zoehed.mh", "fakmir@cebfirvot.pm" ] 
	}
	```
	- 3 index entries created:
		- "Beatrice McBride", "puovvid@wamaw.kp"
		- "Beatrice McBride", "todujufo@zoehed.mh"
		- "Beatrice McBride", "fakmir@cebfirvot.pm"


<br>

	
### GEOSPATIAL INDEXES
```js
// How to create 2d index: 
db.col.createIndex( 
	{ <location field>:"2d", <additional field>:<value> },
	{ <index-specification options> } 
)
```
```js
// How to create a 2dsphere index: 
db.<collection>.createIndex( 
	{ <location field>:"2d", <additional field>:<value> },
	{ <index-specification options> } 
)
```
- How to create geoJSON points for a 2dsphere indexed field in MongoDB
- How to query for geoJSON points:
    -> Within a circle
    -> Near a point
    -> Within a polygon


<br>


### TEXT INDEXES
```js
db.col.createIndex({<field1>:"text", <field2>:"text"})
```
- Text indexes can include any field whose value is a string or an array of string elements. 
- A collection can only have one text search index, but that index can cover multiple fields.

- per buscar fem (buscara tots els camps indexats amb text index):
```js
db.col.aggregate([ {$match: {$text: {$search: "hola"} } } ])
```

- sort results by score
```js
db.blog.createIndex(
	{ content: "text", keywords: "text", about: "text" },
	{
		weights: {
			content: 10,
			keywords: 5
		},
		name: "TextIndex"
	}
)

// field "about" has the default weight of 1
// podem fer un sort de dos maneres:
{$sort: {score: {$meta:"textScore"} }}
{$project: {score:{$meta:"textScore"}}}, {$sort:{score:-1}}  
```
- For each indexed field in the document, MongoDB multiplies the number of matches by the weight and sums the results. Using this sum, MongoDB then calculates the score for the document.


<br>


### HASHED INDEXES
- Hashed indexes maintain entries with hashes of the values of the indexed field.
- db.collection.createIndex( { _id: "hashed" } )
- Compound hash index (only one hash index permitted per compound index):
```js
db.collection.createIndex( { "fieldA" : 1, "fieldB" : "hashed", "fieldC" : -1 } )
```

- It supports Hashed based sharding: use a hashed index of a field as the shard key to partition data across your sharded cluster.


<br>


### WILDCARD INDEXES
- Used for collections with unpredictable queries
- They only cover queries on a single field.
```js
db.createIndex({'$**':1})	// Index everything
db.createIndex({"a.b.$**": 1})	// Index "a.b" and all subpaths
db.createIndex({"$**": 1}, {wildcardProjection: {a: 1}})	// Index "a" and all subpaths
db.createIndex({"$**": 1}, {wildcardProjection: {a: 0}})	// Index everything but "a"
```


<br>


### SORTING WITH INDEXES
Ex: When are we using an index when sorting ?
```js
db.createIndex({a:1, b:1, c:1, d:1})
db.col.find({}).sort(a:1, b:1, c:1, d:1)	// IXSCAN
db.col.find({}).sort(a:1, b:1)	// IXSCAN
db.col.find({}).sort(b:1, a:1)	// COLLSCAN and SORT(in memory sort which is really bad for performance)

db.col.find({"e": "test"}).sort(a:1, b:1, c:1, d:1)	// IXSCAN only for sorting but a COLLSCAN is performed in find stage
db.col.find({a: "test", b:"test"}).sort(c:1, d:1)	// IXSCAN
db.col.find({a: "test"}).sort(c:1, d:1)	// IXSCAN and SORT -> IXSCAN in find stage but in memory sort
```
```js
db.createIndex({a:1, b:-1})
db.col.find({}).sort(a:1)	// IXSCAN
db.col.find({}).sort(a:1, b:-1)	// IXSCAN
db.col.find({}).sort(a:-1)	// IXSCAN
db.col.find({}).sort(a:-1, b:1)	// IXSCAN
// all other possible combinations would result in a COLLSCAN  
```


<br>

		
### EXPLAIN METHOD (cursor.explain())	
- Create an explainable object:
```js
var exp = db.col.explain(); 	
exp.find({});
```
		
- Modes:
	- **queryPlanner**: default mode. Doesn't actually execute the query.
		- parsedQuery
		- winningPlan: contains stages with indexes used, COLLSCAN/IXSCAN
	- **executionStats**: also contains queryPlanner
		- executionSuccess
		- nReturned
		- totalKeysExamined
		- totalDocsExamined
		- executionTimeMillis
		- executionStages: igual que winningPlan pero amb mes info
	- **allPlansExecution**: more verbose (more information about rejectedPlans). Also contains queryPlanner and executionStats.

- *explain()* on a sharded cluster
	- Each shard chooses its own winningPlan
	- last stage is a **SHARD_MERGE** containing all winningPlans for each shard involved in the query


<br>


### SELECTING AN INDEX
- Quan entra una query es seleccionen els index que es poden utilitzar per a satisfer la query shape.
	- Es crea una query Plan per cada index i s'executa la query amb cadascun d'ells.
	- Quin query Plan guanya ?
		- la que retorna tots els resultats primer
		- la que retorna un numero limitat de documents ordenats
	- a query Plan que guanya es guarda en un cache
	- Quan es reexecuta el query Plan ?
		- es produeixen un numero determinat de escriptures
		- si s'inclouen, s'eliminen o s'actualitzen index nous
		- si es reinicia la instancia de mongoDB


<br>


### COVERED QUERIES
- Queries satisfied entirely by index keys
- We only examine index keys which are tipically stored in RAM (.explain() → totalDocsExamined: 0)

- You can't cover a query if...
	- any of the indexed fields are arrays or embedded documents
	- when run against a mongos if the index does not contain a shard key


<br>

 
### INDEXING STRATEGIES
- How to create an index that supports a query that sorts on one field, queries for an exact match on a second field, and does a range query on a third field:
```js
db.col.find({student_id:{$gt:500}, class_id:54}).sort({final_grade:1})
```
```js
db.createindex({student_id:1, class_id:1, final_grade:1})
db.createindex({class_id:1, final_grade:1, student_id:1})
```
- el 2 es més rapid perque primer busquem per un valor class_id especific, amb la qual cosa eliminem molts mes documents que si busquem primer per student_id.
- a més, es posa primer el camp del sort ja que aixi aconseguim que mentre es fa el sort es vagin eliminant els documents que no tinguin student_id mes gran que 500.


<br>


### EFFECTS OF INDEXES ON WRITE PERFORMANCE
- Indexes generally speed up read performance, but can slow down write performance.
	- quan insertem un document necessitem actualitzar els indexes
- Hybrid actions (e.g., with a find and a write) such as update and remove, may depend on the use case (though they usually are faster with indexes).


<br>


### UNIQUE INDEXES
- A unique index ensures that the indexed fields do not store duplicate values
```js
db.col.createIndex( { "a": 1 }, { unique: true } )
```
- Obtenim un Duplicate Key Error
	- Si intentem afegir un element duplicat.
	- Si intentem crear el Unique Index quan existeixen elements duplicats

- Unique compound index: enforce uniqueness on the combination of the index key values.


<br>


### TTL INDEXES
```js
db.col.createIndex( { "lastModifiedDate": 1 }, { expireAfterSeconds: 3600 } )
```
- Automatically remove documents from a collection after a certain amount of time or at a specific clock time
- TTL indexes expire documents after:
	- indexed field value + specified number of seconds.
- If the field is an array, and there are multiple date values in the index, MongoDB uses lowest (i.e. earliest) date value.
- If the indexed field in a document is not a date or an array that holds one or more date values, the document will not expire.
- If a document does not contain the indexed field, the document will not expire.


<br>


### HIDDEN INDEXES
```js
db.addresses.createIndex({ “a”: 1 }, { hidden: true });
```
- Hide/unhide existing index: 
```js
db.col.hideIndex({a:1});  // Or specifying the index name
db.col.unhideIndex({a:1})
```
- By hiding an index from the planner, users can evaluate the potential impact of dropping an index without actually dropping the index.


<br>


### PARTIAL INDEXES
```js
db.col.createIndex(
	{ a: 1, b: 1 },
	{ partialFilterExpression: { a: { $gt:5 } } }
)
```
- Only index a subset of the documents in a collection. This means lower storage requirements (both memory and disk)
- MongoDB will not use the partial index for a query or sort operation if using the index results in an incomplete result set.
```js
db.col.find({ a:{$gte: 10} })  	//Using index
db.col.find({ a:{$lt:10} })		//Not using index
```

- Partial indexes preferred over Sparse Indexes.
	- Greater control over which documents are indexed.
	- A superset of the functionality offered by sparse indexes.

- Restrictions:
	- You cannot specify both the partialFilterExpression option and the sparse option.
	- _id indexes cannot be partial indexes.
	- Shard key indexes cannot be partial indexes.
- Partial Filter with a Unique constraint → the unique constraint only applies to the documents that meet de filter expression.


<br>


### HYBRID INDEX BUILD
- Previous versions of MongoDB supported building indexes in the background or foreground, i.e., blocking reads/writes or not during index build.
- In new versions, index building has been optimized. Es fa sense bloquejar els read/writes.
	- The optimized index build performance is at least on par with background index builds. 
	- For workloads with few or no updates received during the build process, optimized index builds can be as fast as a foreground index build on that same data.
- Building indexes during time periods where the target collection is under heavy write load can result in reduced write performance and longer index builds.


<br>


### REGEX ON STRING BUILDS AND INDEXES
- Regular index on a string field indexes the entire value in the string
	- useful for single word strings where you can match exactly.

- Text index will tokenize and stem the content of the field. So it will break the string into individual words or tokens, and will further reduce them to their stems so that variants of the same word will match ("talk" matching "talks", "talked" and "talking" for example, as "talk" is a stem of all three).
	- Useful for true text (sentences, paragraphs, etc)

- Regex optimization: Utilitzar “prefix expression”, és a dir, començar el regex amb "^" o "\A"
