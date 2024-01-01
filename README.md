# MongoDB The Complete Developer Guide

NoSQl database that enforces no schemas, but doesn't mean that it doesn't exists.

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

These are the principal types in Mongo:

- Text (Only size limited by the document limit that is 16mb);
- Boolean;
- Number;
  - Integer (int32) \[55\];
  - NumberLong (int64) \[10000000000\];
  - NumberDecimal (High precision values) \[12.99\];
  - Doubles (Low precision values after decimal places) (The value is rounded) (prices are a good use case, only 2 decimals);
- ObjectId \[ObjectId("sfasd")\];
- ISODate \[ISODate("2018-09-09")\];
  - Timestamp (Usually only used internally) \[11421532\];
- Embedded Document;
- Array

### Number limitations

- Normal integers (int32) can hold a maximum value of +-2,147,483,647;
- Long integers (int64) can hold a maximum value of +-9,223,372,036,854,775,807

- NumberInt creates a int32 value => `NumberInt(55)`
- NumberLong creates a int64 value => `NumberLong(7489729384792)`
- NumberDecimal creates a high-precision double value => `NumberDecimal("12.99")`

OBS: If you just use a number (e.g. insertOne({a: 1}) in the shell, this will get added as a normal double into the database. The reason for this is that the shell is based on JS which only knows float/ double values and doesn't differ between integers and floats.

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

## Schemas in MongoDB

You can structure your schema by those 3 ways:

- **Chaos**: No schema
- **Between Chaos and SQL World**: Some schema
- **SQL World**: Full schema

Most of the time the strategy used will be or **Between Chaos and SQL World** or **SQLWorld**.

### Schema Validation

- **validationLevel**: The level of validation and restriction of the schema validation;
- **validationAction**: What it will do when and schema validation problem occours

| validationLevel                                            | validationAction                          |
|------------------------------------------------------------|-------------------------------------------|
| Which documents get validated?                             | What happens if validation fails?         |
| string: All inserts & updates                              | error: Throw error and deny insert/update |
| moderate: All inserts & update only from correct documents | warn: Log warning but proceed             |

To add and schema validation

```ssh
db.createCollection("posts", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["title", "text", "creator", "comments"],
      properties: {
        title: {
          bsonType: "string',
          description: "must be a string and is required",
        },
        text: {
          bsonType: "string',
          description: "must be a string and is required",
        },
        creator: {
          bsonType: "objectId',
          description: "must be an objectId and is required",
        },
        comments: {
          bsonType: "array",
          description: "must be an array and is required",
          items: {
            bsonType: "object",
            required: ["text", "author"],
            properties: {
              text: {
                bsonType: "string",
                description: "must be a string and is required"
              },
              author: {
                bsonType: "objectId",
                description: "must be a objectId and is required"
              },
            }
          }
        }
      }
    },
  }
})
```

To add validationActions

```ssh
db.runCommand({
  collMod: "posts",
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["title", "text", "creator", "comments"],
      properties: {
        title: {
          bsonType: "string',
          description: "must be a string and is required",
        },
        text: {
          bsonType: "string',
          description: "must be a string and is required",
        },
        creator: {
          bsonType: "objectId',
          description: "must be an objectId and is required",
        },
        comments: {
          bsonType: "array",
          description: "must be an array and is required",
          items: {
            bsonType: "object",
            required: ["text", "author"],
            properties: {
              text: {
                bsonType: "string",
                description: "must be a string and is required"
              },
              author: {
                bsonType: "objectId",
                description: "must be a objectId and is required"
              },
            }
          }
        }
      }
    },
  },
  validationAction: "warn"
})
```

## How to modelling your database

Use those questions to guide you:

| Question                                                | Type of Data                                       | How it changes the modelling of your database                                            |
|---------------------------------------------------------|----------------------------------------------------|------------------------------------------------------------------------------------------|
| Which Data does my App need or generate?                | User Information, Product Information, Orders, ... | Defines the Fields you'll need (and how they relate)                                     |
| Where do i need my Data?                                | Welcome Page, Products List Page, Orders Page      | Defines your required collections + field groupings                                      |
| Which kind of Data or Information do i want to display? | Welcome Page: Product Names; Products Page: ...    | Defines which queries you'll need                                                        |
| How often do i fetch my data?                           | For every page reload                              | Defines whether you should optimize for easy fetching (duplicating data for easy access) |
| How often do i write or change my data?                 | Orders => Often, Product Data => Rarely            | Defines whether you should optimize for easy writing                                     |

## Relations

How relations works in MongoDB

### Forms to create relations

| Nested/Embedded Documents                                                                      | References                                                                                      |
|------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|
| Group data togheter logically                                                                  | Split data across collections                                                                   |
| Greate for data that belongs together and is not really overlapping with other data            | Great for related but shared data as well as for data which is used in relations and standalone |
| Avoid super-deep nesting (100+ levels) or extremely long arrays (16mb size limit per document) | Allows you to overcome nesting and  size limits (by creating new documents)                     |

### Examples of relations type and forms

| Relation Type | Relation form | Example                                                                           |
|---------------|---------------|-----------------------------------------------------------------------------------|
| One to One    | Embedded      | One patient has one disease summary, a disease summary belongs to one patient     |
| One to One    | References    | One person has one car, a car belongs to one person                               |
| One to Many   | Embedded      | One thread has many answers, one answer belongs to one question thread.           |
| One to Many   | References    | One city has many citizens, one citizen belongs to one city                       |
| Many to Many  | Embedded      | One customer has many products (via orders), a product belongs to many customers. |
| Many to Many  | References    | One book has many authors, an author belongs to many books.                       |

### Joining collection to search relations

You can search like that using `$lookup` in the `aggregate` method

Collections data

```json
{
  "customersCollection": [
    {
      "userName": "max",
      "favBooks": ["id1", "id2"]
    }
  ],
  "booksCollection": [
    {
      "_id": "id1",
      "name": "Lord of the Rings 1",
    }
  ]
}
```

Aggregate method with lookup

```ssh
db.customers.aggregate([
  {
    $lookup: {
      from: "books"
      localField: "favBooks",
      foreignField: "_id",
      as: "favBookData"
    }
  }
])
```

## Working with Shell and Server


