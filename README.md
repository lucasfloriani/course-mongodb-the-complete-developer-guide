# MongoDB The Complete Developer Guide

## Common Commands

- "cls": Clear the terminal text;
- "show dbs": Show all databases in the mongo;
- "use database-name": Connects to the database name, creates it if don't exists when you inserts data;
- "db.nameOfCollection.insertOne()": Creates a collection in the current database connected and insert one data;
- "db.nameOfCollection.find()": Returns the data stored in this collection;
- "db.nameOfCollection.find().pretty()": Returns the data stored in this collection but in a more readable way.

## MongoDB Structure Related to Relational Databases

- Database == Database
- Collection == Table
- Document == Table Row

OBS: Collection and Documents are created implicitly/automatically without needing to define it.

## Data Types

- ObjectId;
- String;
- Number;
- Boolean

### Embedded Documents

A field in the document can be another document (nested documents).

OBS: Up to 100 levels of nesting.
OBS2: Document size can be at limit of 16mb.
OBS3: Can have an array of embedded documents (arrays can hold ANY data).

## Crud Operations

- Create
  - insertOne(data, options)
  - insertMany(data, options)
- Read
  - find(filter, options)
  - findOne(filter, options)
- Update
  - updateOne(filter, data, options)
  - updateMany(filter, data, options)
  - replaceOne(filter, data, options)
- Delete
  - deleteOne(filter, options)
  - deleteMany(filter, options)

OBS: **update** can be used without $set, but it will overwrite all content by the new update object.
OBS2: **update** is not a recomended way to overwrite, use **replaceOne** instead

### Acessing Structured Data

```ssh
# Get hobbies array
db.passengers.findOne({ name: "Albert Twostone" }).hobbies

# Find users by structured data
db.passengers.find({ hobbies: "sports" }).pretty()

# Get by nested document
db.flightData.find({ "status.description": "on-time" }).pretty()

# Get by even more nested document
db.flightData.find({ "status.details.responsible": "Max" }).pretty()
```

## Filter

**find** don't return an array of all documents from a collection, but it return a cursor object with a lot of metadata behind it, an is used to cycle through the results.

**find().toArray()** transforms all documents from this cursor to an array, but is not optimized when the collection has a big amount of registers.

**find().forEach((passengerData) => { printfjson(passengerData) })** used to transform data in the find method (Better when has a big amount of registers) (Get data by demand, not return all data at once).

OBS: By default it will return the first 20 documents in the collection and stops.
OBS2: Only find return a cursor, findOne don't use a cursor and it is the cause that using with pretty() return an error (pretty is a method from a cursor).

## Projection

Is a way to get only the field in the document that is needed.
Can be set in the find method as the second parameter.

```ssh
db.passengers.find({}, { name: 1, _id: 0 }).pretty()
```

OBS: \_id is always included, need to set like this to don't return it { \_id: 0 }.
