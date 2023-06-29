## Java driver

### Creating a MongoClient instance
```java
ConnectionString conString = new ConnectionString(MONGO_URL);
MongoClientSettings settings = MongoClientSettings.builder()
    .applyConnectionString(conString)
    .serverApi(ServerApi.builder()
        .version(ServerApiVersion.V1)
        .build())
    .build();
MongoClient mongoClient = MongoClients.create(settings);
```

- An application should use single MongoClient instance (we can use a Singleton design pattern)
- Instantiating MongoClient is resource intensive

- The Document class is a representation of a BSON document
    - BSON is a data format optimized for storage, retrieval and transmission

<br>

### Helper builder classes
- Filters
- Aggregates
    - Match
    - Project
    - Group
    - Sort
- Sorts

<br>

### Mongoshell commands
```js
insertOne(doc)
insertMany(doc)

updateOne(filter, update, options)
updateMany(filter, update, options)
replaceOne(filter, doc, options)

findAndModify() //find a document, return it and update it
findOneAndDelete() //find document, return it and delete it
find()
aggregate()

deleteOne()
deleteMany()

getIndexes()
dropIndex()
dropIndexes()

drop()
```

<br>

### Indexes
db.coll.getIndexes()

- Multikey
    - support efficient queries against array fields by creating an index key for each element in the array. 
    - This allows MongoDB to search for the index key of each element in the array rather than scan the entire array, which results in dramatic performance gains in your queries.
    - only allows one field that is an array

- Compound indexes
    - An index that contains references to multiple fields within a document
    - The recommended order of indexed fields in a compound index is Equality, Sort, and Range. Optimized queries use the first field in the index, Equality, to determine which documents match the query. The second field in the index, Sort, is used to determine the order of the documents. The third field, Range, is used to determine which documents to include in the result set.

<br>

- Deleting an index that is the only index supporting a query will affect the performance of that query. You should hide the index before deleting it. This way, you'll be able to assess the impact of removing the index on query performance. MongoDB does not use hidden indexes in queries but continues to update their keys. This allows you to assess if removing the index affects the performance and unhide the index if needed. Unhiding an index is faster than recreating it.

<br><br>

## Data modeling

- Principles:
    - data that is accessed together should be stored together
    - avoid unbounded documents. Ex: a collection of blog posts that contains an array of comments made by people that see the blogpost; the array might grow too large, it might be better to store the comments in a separate collection

### Embedding vs referencing
- Embedding:
    - single query to retrieve data
    - single operation to update/delete data
    - !! large documents
    - !! data duplication
- Referencing
    - no duplication
    - smaller documents
    - !! might need to join data from multiple documents

<br><br>

## Atlas

### Search index
- Define relevance-based search. Based on apache Lucene.
- usage: 
```js
db.restaurants.aggregate([
    {"$search": {"text": {
        "path": "name", 
        "query": "cuban"
    }}}
])
```

- Options:
    - Index analyzer and Search analyzer
        - 40 plus index analyzers
        - how will we tokenize the data into searchable terms
    - Dynamic mapping
        - all fields are indexed (except for booleans, objectIds and timestamps)
    - Field mappings
        - define data types and input parameters for specific fields

- Autocomplete index
    - tokenization types:
        - **edgeGram** look for matches at the beginning of a word on the name field
        - **nGram** 
        - **regexCaptureGroup**