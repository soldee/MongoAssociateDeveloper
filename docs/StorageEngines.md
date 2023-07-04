# Storage engines

- The storage engine is the component of the database that is responsible for managing how data is stored, both in memory and on disk. MongoDB supports multiple storage engines, as different engines perform better for specific workloads.

### WIRED TIGER
- Document-level concurrency
    - Multiple clients can modify different documents of a collection at the same time
    - When a conflict between two operations is detected, one will incur a write conflict causing MongoDB to transparently retry that operation.

- Compression
    - Minimize storage use at the expense of additional CPU
    - With WT, MongoDB supports compression for all collections and indexes. 
    - snappy compression by default. zlib and zstd are also available (these provide higher compression with higher CPU usage)

- Checkpoints
    - Act as recovery points. 
    - Created every 60 seconds
    
- Journal
    - In combination with checkpoints it ensures data durability
    - WT journal persists all data between checkpoints so that if MongoDB exits, it can use the journal to replay all data modified since the last checkpoint.

- With journaling, the recovery process looks like:
    1. Looks in the data files to find the identifier of the last checkpoint
    2. Search in the journal files for the record that matches the identifier of the last checkpoint
    3. Apply the operations in the journal file since the last checkpoint