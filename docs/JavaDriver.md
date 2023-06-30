# Java driver

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
