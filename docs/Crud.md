# CRUD

## CREATE
db.collection.insert(<document or array of documents>, { writeConcern: <document>, ordered: <boolean> } )
	-> Ex: db.test.insert([{"_id":1, "test":1}, {"_id":1, "test":2}, {"_id":2, "test":3}], {ordered:false})
		# s'inserten test 1 i test 3
	-> Ex: db.test.insert([{"_id":1, "test":1}, {"_id":1, "test":2}, {"_id":2, "test":3}])   //ordered: true by default
		# s'inserta test 1 nomÃ©s (ja que desprÃ©s salta un error)
		
db.collection.save(<document>)
	-> If the document does not contain an _id, then the save() method calls the insert() method.
	-> If the document contains an _id, then the save() method is equivalent to an update with the upsert and the query predicate on the _id field.

db.collection.update(query, update, options)
	-> UPSERT option: update if document exists or insert if it doesn't
	
db.collection.findAndModify()
	-> Modifies and returns a single document.
	-> By default, the returned document does not include the modifications made on the update.
	-> To return the document with the modifications made on the update, use the new option.

	-> ex: db.test.update({test:3}, {$set: {a:1}}, {upsert:true}) //suppose there is no record with test:3
		# the query inserts a new document: {_id:ObjectId("09c4mmc2m3"), test:3, a:1} //inserta tambe el field de la query
		

bulkWrite() vs insertMany()
- they are the same performance wise
- bulkWrite is only executed when we call bulk.execute()
- bulkWrite allow you to have a mix of updates/deletes or inserts


ordered vs unordered bulk
- In an ordered bulk, if an error occursin a write operation, MongoDB will return without processing any remaining write operations in the list.
	-> in an unordered, the operation continues
- Unordered bulk is generally faster


<br><br>


## READ
- Matching arrays //tenim un document {"_id" : ObjectId("620b7ecdf8a2562f2c771222"),"cast" : [ "Uri", "Biel", "Solde"]}
	-> db.col.find({cast: ["Uri"]})  //sense resultats
	-> db.col.find({cast: "Uri"})	 //match
	-> db.test.find({cast: {$in: ["Uri", "Bi"]}})   //match
	-> db.test.find({cast: {$all: ["Uri", "Biel"]}})	//match
	-> db.test.find({cast: {$all: ["Uri", "Bi"]}})	//sense resultats
	
- $size	-> Ex: db.test.find({cast: {$size: 3}})
- $elemMatch -> util quan tenim array amb molts objectes: [{}, {}, {}]

OPERATORS
$lt, $gt, $gte, $lte, $eq, $neq	
$and, $or, $nor(fail to match both given clauses), $not		// { $and: [{}, {}, ...] }


- $expr 	// allows the use of aggregation expressions within the query language
Ex: 	-> db.getCollection('test').find({$expr: {$eq: ["NEW", "$a"] }}).count()
 (igual)-> db.getCollection('test').find({"a": {$eq: "NEW" }}).count()
 
- $type  //look for documents that have a particular value type for a field
		-> Ex: db.getCollection('test').find({"a": {$type: "array"}})

- $exists  -> check if field exists -> Ex: db.getCollection('test').find({"a": {$exists: true}})
		   -> it does not match fields set to null, it just checks the existence of the field
				#to match non existing fields and fields set to null -> db.getCollection('test').find({"a": null})
				
- db.col.distinct(field, query, options)   -> Finds the distinct values for a specified field across a single collection or view and returns the
											  results in an array.
	-> Ex: db.getCollection('test').distinct("a")
(igual)	-> db.getCollection('test').aggregate([{$group: { _id: "$a" }}])


<br><br>


## UPDATE
db.col.update(<query>, <update>, <options>)

- Update Operators
{ $set: { <field1>: <value1>, ... } }
$currentDate: Sets the value of a field to current date, either as a Date or a Timestamp.
$inc: Increments the value of the field by the specified amount.
$min: Only updates the field if the specified value is less than the existing field value.
$max: Only updates the field if the specified value is greater than the existing field value.
$mul: Multiplies the value of the field by the specified amount.
$rename: Renames a field.
$set: Sets the value of a field in a document.
$setOnInsert: Sets the value of a field if an update results in an insert of a document
$unset: Removes the specified field from a document

- Array update operators:
$addToSet: Adds elements to an array only if they do not already exist in the set.
$pop: Removes the first or last item of an array.
$pull: Removes all array elements that match a specified query.
$push: Adds an item to an array.
$pullAll: Removes all matching values from an array.

- $ (update)
The positional $ operator identifies an element in an array to update without explicitly specifying the position of the element in the array.
db.collection.updateOne(
   { <array>: value ... },
   { <update operator>: { "<array>.$" : value } }
)


<br><br>


## DELETE
- Drop collection: db.col.drop()
-remove documents: db.col.deleteOne(<query>)

- TTL (Time To Live)
	-> Expire Documents after a Specified Number of Seconds
		# db.log_events.createIndex( { "createdAt": 1 }, { expireAfterSeconds: 10 } )
		# db.log_events.insertOne( {"createdAt": new Date(), "logEvent": 2,"logMessage": "Success!"} )
			#el document s'eliminarÃ  automÃ ticament en 10 segons
	-> Expire Documents at a Specific Clock Time
		# db.log_events.createIndex( { "expireAt": 1 }, { expireAfterSeconds: 0 } )
		# db.log_events.insertOne( {"expireAt": new Date('July 22, 2013 14:00:00'),"logEvent": 2,"logMessage": "Success!"} )
			#especifiquem el expireAt en cada document que creem per dir quan s'ha d'eliminar automaticament
