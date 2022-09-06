+++
title = "MongoDB"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "alain.bouchard@appdirect.com"
disableToc = "false"
+++

{{< toc >}}

## MongoDB

show dbs
show collections

javascript shell

- `use <db name>`
- `db.getName()` -> will return current DB

### Create a doc

- `doc = {"key":value, ...}`

### Insert to a collection

- `db.<collection>.insertOne(doc)`

### Show documents

- `db.<collection>.find()`
- `db.<collection>.find().pretty()` -> json file formated
- `db.<collection>.find({}, {"title": 1})` -> will return only the title for all documents
- `db.<collection>.find({"title": "tacos"})` -> will return all documents that match the condiction, ex. "title":"tacos"

### Searching using regex

`db.<collection>.find("title": {$regex: /taco/i}}, {title: 1})` -> returns the title for all documents that title march regex /taco/i

### Sort, limit and skip

- `db.<collection>.find().count()`
- `db.<collection>.find({}, {"title": 1}).sort({"title": 1})` where 1 for asc and -1 for desc
- `db.<collection>.find().limit(2)` -> returns 2 results only
- `db.<collection>.find({}, {"title": 1}).skip(1)` -> will return the items skipping first one

### Operators and arrays

```text
greater than: $gt
less than: $lt
less than or equal to: $lte
```

- `db.<collection>.find({ "cook_time": { $lte: 30 }}, {"title":1})` -> find items with cook_time <= 30
- `db.<collection>.find({ "cook_time": { $lte: 30 }, "prep_time": { $lte: 10 } },  {"title":1})` -> find items with cook_time <= 30 *AND* prep_time <= 10
- `db.<collection>.find( { $or: [{ "cook_time": { $lte: 30 }, "prep_time": { $lte: 10 } }]},  {"title":1})` -> find items with cook_time <= 30 *OR* prep_time <= 10

```text
$all operator: require all items in a doc arrays
$in operator: require one from the arrays
```

- `db.<collection>.find( { "tags" : {$all: ["easy", "quick"]}}, {"title":1, "tag":1})` -> returns item with both tags "easy" or "quick"
- `db.<collection>.find( { "tags" : {$in: ["easy", "quick"]}}, {"title":1, "tag":1})` -> returns item with either tags "easy" or "quick"

### Updating a document

```text
$set
$unset
$inc -> increment
```

### Update one field

- `db.<collection>.updateOne(find, modification)`
- `db.<collection>.updateOne({"title": "pizza"}, { $set: {"title":"thin crust pizza"}})`

### Add one field

- `db.<collection>.updateOne({"title": "pizza"}, { $set: {"vegan":true}})`

### Remove one field

- `db.<collection>.updateOne({"title": "pizza"}, { $unset: {"vegan":1}})` -> one for true?!?!

### Increment a counter by N

- `db.<collection>.updateOne({"title": "pizza"}, { $inc: {"likes_count":1}})` -> increment likes_count field by 1

### updating an array of a document

- `db.<collection>.updateOne({"title": "pizza"}, { $push: {"likes":60}});` -> will add value 60 to the likes array
- `db.<collection>.updateOne({"title": "pizza"}, { $pull: {"likes":60}});` -> will remove value 60 from the likes array

### Delete a document

- `db.<collection>.deleteOne({"_id":"...."})`
