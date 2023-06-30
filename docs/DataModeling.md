# Data modeling

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