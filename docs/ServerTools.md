# SERVER TOOLS

### mongoexport
- cli tool that produces a JSON/CSV export of data stored in a MongoDB instance
- If you do not specify a file to write, it writes to standard output
- If you need to preserve all BSON data types to perform a full instance backup, specify **--jsonFormat=canonical**
```js 
url="mongodb://user:pwd@host:port/db"
fields="ID,valor"
mongoexport --uri ${url} --collection="x" --fields=${fields} --type="csv" --out="result.csv"
```


<br>


### mongoimport
- cli tool to import content from an Extended JSON, CSV, TSV export
- automatically detects file format
```js
url="mongodb://user:pwd@host:port/db"
mongoimport –uri ${url} --collection docs.chunks --numInsertionWorkers 4 --mode upsert chunks
```


<br>


### mongostat
- provides a quick overview of the status of a currently running mongod or mongos instance (every second by default).
```js
mongostat <options> <connection-string> <polling interval in seconds>
```
- Retorna un output similar al següent:
![Mongostat output](img/mongostat_output.PNG)

- Les opcions més interessants són:
    - Per MMAPv1 storage engine hi ha la opció “faults” que és un indicador del IO. 
    - Per WiredTiger podem mirar el “dirty”, “used” per veure quanta memoria estem utilitzant.


<br>


### mongotop
- track the amount of time a mongod instance spends reading/writing data with statistics on a per-collection level (every second by default).
- mongotop [options] [frequency]
