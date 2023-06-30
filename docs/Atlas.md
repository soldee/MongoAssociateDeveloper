# Atlas

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