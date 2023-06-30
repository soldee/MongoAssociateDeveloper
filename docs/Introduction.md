# Introduction

### MongoDB vs SQL
- Data stored in documents instead of tables
    - each document can be completely different
- MongoDB is designed for scalability

<br>

### Architectural Components that comprise MongoDB
- MQL (MongoDB Query Language)
    - set of instructions and commands to interact with MongoDB
    - translate incoming BSON wire protocol messages into mongoDB operations

- MongoDB Data Model Layer 	
    - responsible for applying all the CRUD operations defined in the MQL
    - management of namespaces, indexes, data structures
	- replication mechanisms: write/read concerns

- Storage Layer -> how data is stored on disk, what kind of files does it used, levels of compression, ...

- Security	-> user management, authentication, encryption

- Admin	-> server administration (creating DB, renaming collections, logging infrastructure, ...)


<br>


### REPLICASET (groups of different mongod)
- Primary node managing all read/write ops
- Secondary nodes that copy the info from the primary node
- Purpose is to ensure:
    - High Availability
	- Automatic Failover (in case of failure, a secondary node acts as a primary node automatically)


<br>


### VERTICAL/HORIZONTAL SCALING
- Vertical Scaling involves increasing the capacity of a single server (+CPU, +RAM, +storage space)

- Horizontal Scaling involves dividing the system dataset and load over multiple servers, adding additional servers to increase capacity as required
    - each machine handles a subset of the overall workload
	- The trade off is increased complexity in infrastructure and maintenance for the deployment.


<br>


### SHARDED CLUSTER
- Composed of:
	- shards: each shard contains a subset of the sharded data. Each shard can be deployed as a replica set.
	- mongos: acts as a query router, providing an interface between client applications and the sharded cluster
	- Config Servers: store metadata and configuration settings for the cluster


<br>


### BSON
- MongoDB stores data as BSON
- BSON is
	- lightweight: space required to represent data is kept to a minimum
	- traversable:
	- efficient: encoding/decoding to/from BSON is very quick
- BSON extends the JSON value types (provides byte, Int32, Date, ...)
- MongoDB drivers
	- send and receive data as BSON
	- map BSON to whatever data types are most appropiate for each programming language


<br>


### MONGOSH
```js
show collections
show dbs
use <DBname>
	
save(<validJSONdocument>)
update(<match>, <update>, <options>)

insertOne(<doc>)
insertMany(<doc>)

updateOne(<filter>, <update>, <options>)
updateMany(<filter>, <update>, <options>)
replaceOne(<filter>, <update>, <options>)

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