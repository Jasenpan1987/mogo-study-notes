## 1. Basic shell command

```
show dbs // list all the available dbs

use shop // switch to or create a new db called "shop"
```

```
db.products.insertOne({ name: "A Book", price: 2.99 })
```

create a collection and insert the json into it.

```
db.products.find()
```

will list all the available record in the products collection.

1. Mongo is schema-less, so we don't have to keep the same schema for each record.

2. Each field can be a nested field, which means it can also store an object.

## 2. CRUD and basic data types

```
db.flightData.insertOne({
     "departureAirport": "MUC",
     "arrivalAirport": "SFO",
     "aircraft": "Airbus A380",
     "distance": 12000,
     "intercontinental": true
   })
```

Every document that inserted to mongodb will automatically add a "\_id" property to it.

In Mongodb, two documents in one collection can have different schemas.

When you insert a document, you can give the \_id by yourself as long as you can ensure the uniqueness.

### 2.1 CRUD quries

- Create:

```
insertOne(data, options)

insertMany(data, options)
```

- Read:

```
find(filter, options)
findOne(filter, options)
```

- Update

```
updateOne(filter, data, options)
updateMany(filter, data, options)
replaceOne(filter, data, options)
```

- Delete

```
deleteOne(filter, options)
deleteMany(filter, options)
```

- When updating data, you need to use `$set` key to wrap your changes. And if the field is exist, it will just update the value, otherwise it will add the field.

```
db.flightData.updateMany({$set: {marker: "updated"}})
```

```
db.flightData.updateMany({}, {$set: {marker: "updated"}})
```

```
db.flightData.insertMany([
   {
     "departureAirport": "MUC",
     "arrivalAirport": "SFO",
     "aircraft": "Airbus A380",
     "distance": 12000,
     "intercontinental": true
   },
   {
     "departureAirport": "LHR",
     "arrivalAirport": "TXL",
     "aircraft": "Airbus A320",
     "distance": 950,
     "intercontinental": false
   }
 ])
```

### 2.2 find filters

When we do `find` opertation in mongodb, we can pass in a filter liked object, just like this:

```
db.flightData.find({distance: 12000})
```

For numbers like distance, we can query the documents which has distance greater than 10000 like this:

```
db.flightData.find({distance: {$gt: 10000}})
```

And for the above examples, if we use `findOne` it will only bring back the first matching document.

### 2.3 Update vs updateMany

UpdateOne and updateMany do not accept plain object as the updater document, in another word, you have to the \$set operator. But update does accept the plain object, however, it will replace the document with the update entirely.

To be safe, do not use update when you want to update some fields in one or many document. And if you indeed want to replace the document, you can use `replaceOne`

### 2.4 find in detail

`find` does not give us back a list of documents as we expected, it gives us a cursor, which has some metadata in it. And if we want to have the data, we need to call `toArray()` at the end of the `find` query.

The reason that we can see the data when we execute the `find` in the shell is because mongo shell give us the ability to see the first documents in the cursor.

### 2.5 Projection

When you try to fetch the data from mongodb server, say the data structure is like this:

```
{
  name: "foo",
  age: 20,
  email: "foo@123.com",
  ...
}
```

however, in your application, you only care about the name and age, not the other fields, to save some bandwidth, it's better to do this from the mongodb side.

```
db.passengers.find({}, {name: 1, age: 1})
```

here, the second object in find is the projection, the key is the field key in the documents, and for values, 1 mean include 0 means not include.

By default, \_id is always included. You have to say `{_id: 0}` to exclude it.

### 2.6 Embedded documents

```
{
  id: 1,
  age: 10,
  name: {
    firstName: "Jasen",
    lastName: "Pan"
  }
}
```

In mongodb, you can have fields which is also a sub document. And you can have up to 100 level of nesting, and each document can up to 16mb in size.

Also, you can have arrays in mongodb like this

```
{
  id: 1,
  age: 10,
  name: {
    firstName: "Jasen",
    lastName: "Pan"
  },
  hobbies: ["play", "eat"]
}
```

Also, mongo is very similar to javascript, you can access the field on a document by using the dot notation.

```
db.passengers.findOne({ name: "Albert Twostone"}).hobbies
```

When you try to query the hobbies, you can give one value and mongodb is smart enough to search that value in the array.

```
db.passengers.findOne({ hobbies: "cooking" })
```

will give you back the document you need.

To query data inside a sub document, you can also use the dot notation, but you need to wrap the key inside a pair of "".

```
db.flightData.find({ "status.description": "on time" })
```

## 3. Data Modelling

Although Mongodb is schema-less database, which means documents can be totally different, in reality, we need some schema or structures for our applications.

Normally, we need to define some fields which are the core fields, and every document should have it. And some optional fields that designed for a portion of documents. However, if you want the data more structured, you can keep those optional fields to `null`, which is a valid value in mongodb.

### 3.1 Data types

- text: `"jasen"`
- boolean: `true` or `false`
- number:
  - Integer (int32)
  - NumberLong (int64)
  - NumberDecimal
- ObjectId: `ObjectId("asdf")`
- ISODate: `ISODate(2019-01-14)`
- Timestamp: `Timestamp(123333212)`
- Embedded document: `{xxx: xxx}`
- Array: `[1, 2, 4]`

example:

```javascript
db.companies.insert({
  name: "Fresh Apple Inc",
  isStartup: true,
  employees: 33,
  funding: 12345678900987654321,
  details: { ceo: "Foo Bar" },
  tags: [{ title: "apple" }, { title: "fresh" }],
  fundingDate: new Date(),
  insertedAt: new Timestamp()
});
```

### 3.2 Relations

In mongodb, there are two ways to build relations of data

- Nested / Embedded documents
- Reference

For repations we have

- One to one: we normally use embedded documents approach.

- One to many: we normally use embedded documents approach.

- Many to many: we normally use reference approach.

Pros and Cons for each approach:

- Embedded documents: it can get all the data at once, no more lookup required. But it creates duplication and if we need to modify one pice of data, we need to modify in all the places.
- Reference: it saves spaces if the data need to be referenced multiple places and you only need to modify it once. But it requires lookups which need addtional calculation.

There is no gurenteed for which relation to use which approach, it all depends on the application needs of the data.

### 3.3 Join with \$lookup

```javascript
[
  { _id: ObjectId("5c3ca9f8493f137788c46639"), name: "John", age: 35 },
  { _id: ObjectId("5c3caa01493f137788c4663a"), name: "Bill", age: 25 }
][
  ({
    _id: ObjectId("5c3caa37493f137788c4663b"),
    title: "sample book 1",
    authors: [
      ObjectId("5c3caa01493f137788c4663a"),
      ObjectId("5c3ca9f8493f137788c46639")
    ]
  },
  {
    _id: ObjectId("5c3caa53493f137788c4663c"),
    title: "sample book 2",
    authors: [ObjectId("5c3caa01493f137788c4663a")]
  })
];
```

To get the books with the authors data, we can do this

```javascript
db.books.aggregate({
  $lookup: {
    from: "authors", // the outter collection
    localField: "authors", // what does it called in books
    foreignField: "_id", // which field do we want to lookup in author collection
    as: "bookAuthors" // alias
  }
});
```

The result is

```js
[
  {
    _id: ObjectId("5c3caa37493f137788c4663b"),
    title: "sample book 1",
    authors: [
      ObjectId("5c3caa01493f137788c4663a"),
      ObjectId("5c3ca9f8493f137788c46639")
    ],
    bookAuthors: [
      {
        _id: ObjectId("5c3ca9f8493f137788c46639"),
        name: "John",
        age: 35
      },
      {
        _id: ObjectId("5c3caa01493f137788c4663a"),
        name: "Bill",
        age: 25
      }
    ]
  },
  {
    _id: ObjectId("5c3caa53493f137788c4663c"),
    title: "sample book 2",
    authors: [ObjectId("5c3caa01493f137788c4663a")],
    bookAuthors: [
      {
        _id: ObjectId("5c3caa01493f137788c4663a"),
        name: "Bill",
        age: 25
      }
    ]
  }
];
```

### 3.4 Schema Validation

In mongodb, if we want to give the inserted data some validations, we can do that by defining a Schema Validation.

To define a Schema Validation, we need to do it at the very beginning when we create the collection. Instead of creating the collection automatically (`db.users.insertOne(...)`), we need to define it manully.

```js
db.createCollection("posts", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["title", "text", "creator", "comments"],
      properties: {
        title: {
          bsonType: "string",
          description: "must be a string and is required"
        },
        text: {
          bsonType: "string",
          description: "must be a string and is required"
        },
        creator: {
          bsonType: "objectId",
          description: "must be an objectid and is required"
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
                description: "must be an objectid and is required"
              }
            }
          }
        }
      }
    }
  }
});
```

Also, we can configure the action the mongodb takes when an insertion or an update failed due to the violation of the schema validator. By default, if the validation fails, mongodb will raise an error and will prevent the insertion, but we can change this behavior to just give a warning.

```js
db.runCommand({
  collMod: "posts",
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["title", "text", "creator", "comments"],
      properties: {
        title: {
          bsonType: "string",
          description: "must be a string and is required"
        },
        text: {
          bsonType: "string",
          description: "must be a string and is required"
        },
        creator: {
          bsonType: "objectId",
          description: "must be an objectid and is required"
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
                description: "must be an objectid and is required"
              }
            }
          }
        }
      }
    }
  },
  validationAction: "warn"
});
```

## 4. Create in details

- `insertOne({ field: value })`
- `insertMany([{field: value}, {field: value}])`
- `insert()`: Not recomended to use.
- `mongoimport`

### 4.1 Ordered Insert

Let's say we have to insert some hobbies into our hobbies collection and we specify each of their \_ids.

```js
db.hobbies.insertMany([
  { _id: "play", name: "play" },
  { _id: "cooking", name: "cooking" }
]);
```

and now we need to insert three more

```js
db.hobbies.insertMany([
  { _id: "drinking", name: "drinking" },
  { _id: "cooking", name: "cooking" },
  { _id: "fishing", name: "fishing" }
]);
```

Now, the `cooking` one has duplicated \_id with the previous inserted records, it will fail obviously. But it will insert the first record which is "drinking" and will not insert the last "fishing" record. This is because mongodb's `insertMany` operation has an ordered insert by default, to override it, we can do this:

```js
db.hobbies.insertMany(
  [
    { _id: "drinking", name: "drinking" },
    { _id: "cooking", name: "cooking" },
    { _id: "fishing", name: "fishing" }
  ],
  { ordered: false }
);
```

Now it will insert the first one and the last one.

### 4.2 writeConcern

When we write / update data in mongodb, there is an option that we can pass indicates some level of mongodb server's options:

```js
db.hobbies.insertMany(
  [
    { _id: "drinking", name: "drinking" },
    { _id: "cooking", name: "cooking" },
    { _id: "fishing", name: "fishing" }
  ],
  {
    writeConcern: {
      w: 1, // 1 means server need to be acknoledged
      j: true // journal will write the record after inserted
    }
  }
);
```

### 4.3 import data

```
mongoimport ./tv-shows.json -d movieData -c movies --jsonArray --drop
```

- You need to exit your mongo shell to execute this command.
- -d: database name
- -c: collection name
- --jsonArray: need to have it if your json file is in array shape
- drop: drop the collection if it already exist. It will append the data to the end if without this option

## 5. Read in detail

`findOne`, returns first matching record (not the cursor) where `find` returns the cursor.

### 5.1 Operators

#### 5.1.1 Compare operators

```
db.movies.find({runtime: {$gt: 60}})
```

gives back all movies which runtime is GREATER than 60.

```
db.movies.find({runtime: {$ne: 60}})
```

gives back all movies which runtime is NOT EQUAL to 60.

#### 5.1.2 Comparison Operators

```
db.movies.find({"rating.average": {$gte: 7}})
```

searching for movies which rating's average property is greater or equal to 7.

```
db.movies.find({ genres: "Drama" })
```

searching for movies if the "Drama" inside the genre array.

```
db.movies.find({ genres: ["Drama"] })
```

searching for movies if the genre is ["Drama"] only.

```
db.movies.find({ runtime: { $in: [30, 42, 60] } })
```

searching for movies if the runtime is either 30, 42 or 60.

```
db.movies.find({ runtime: { $nin: [30, 42, 60] } })
```

searching for movies if the runtime is NOT 30, 42 or 60.

#### 5.1.2 Logical Operators

```
db.movies.find({"rating.average": {$lt: 5}})
```

searching for movies which has average rating less than 5

```
db.movies.find({"rating.average": {$gt: 9.3}})
```

searching for movies which has average rating more than 9.3

```js
db.movies.find({
  $or: [{ "rating.average": { $lt: 5 } }, { "rating.average": { $gt: 9.3 } }]
});
```

searching for movies which average rating is less than 5 or more than 9.3

```js
db.movies.find({
  $nor: [{ "rating.average": { $lt: 5 } }, { "rating.average": { $gt: 9.3 } }]
});
```

searching for movie which average rating is neither less than 5 nor greater than 9.3

```js
// old syntax
db.movies.find({
  $and: [{ "rating.average": { $gt: 9 } }, { genres: "Drama" }]
});

// new syntax
db.movies.find({ "rating.average": { $gt: 9 }, genres: "Drama" });
```

find the movies which has rating greater than 9 and its genre contains "Drama".

Note: if you want to querry the same field, like this

```js
db.movies.find({ genres: "Horror", genres: "Drama" });
```

this syntax will cover the movies with Horror (the later one)

To avoid this, you need to use the old syntax, which only gives you Drama and Horror.

```js
db.movies.find({ runtime: { $not: { $eq: 60 } } });
```

Searching for movies which has runtime is not equal to 60. We can achieve the same result by using `db.movies.find({runtime: {$ne: 60}})`

#### 5.1.3 Element Operators

```js
db.users.insertMany([
  {
    name: "John",
    phone: 32131232,
    hobbies: [
      { title: "sport", frequency: 4 },
      { title: "cooking", frequency: 6 }
    ]
  },
  {
    name: "Bill",
    phone: "421098223",
    age: 50,
    hobbies: [
      { title: "MTG", frequency: 3 },
      { title: "pokemon", frequency: 7 }
    ]
  }
]);
```

```js
db.users.find({ age: { $exists: true } });
```

searching for user which has an age field, the null value also count. If we want to search for users which has an age field and its value is not null, we can use the following

```js
db.users.find({ age: { $exists: true, $ne: null } });
```

```js
db.users.find({ phone: { $type: "number" } });
```

searching for users who has a phone number which type is a number.

```js
db.users.find({ phone: { $type: ["number", "string"] } });
```

searching for users who has a phone number which type is a number or a string.

```js
db.movies.find({ summary: { $regex: /musical/ } });
```

searching for movies which summary has the word "musical"

#### 5.3.2 \$expr

Let's say we have the following data

```js
[
  { volumn: 100, target: 120 },
  { volumn: 89, target: 89 },
  { volumn: 200, target: 177 }
];
```

```js
db.sales.find({ $expr: { $gt: ["$volumn", "$target"] } });
```

searching for sales which volumn is greater than target

```js
db.sales.find({
  $expr: {
    $gt: [
      {
        $cond: {
          if: { $gt: ["$volumn", 190] },
          then: { $subtract: ["$volumn", 30] },
          else: "$volumn"
        }
      },
      "$target"
    ]
  }
});
```

Searching for sales which volumn is greater than target, and if the sales is greater than 190, it need to subtract 30 then greater than target.

```js
db.sales.find({
  $expr: {
    $gt: [{ $subtract: ["$volumn", "$target"] }, 10]
  }
});
```

Searching for sales which volumn subtract target is more than 10.

#### 5.3.3 Arrays

if we have the following data in our database:

```json
{
  "users": [
    {
      "name": "John",
      "age": 18,
      "hobbies": [
        { "title": "sports", "feq": 2 },
        { "title": "cooking", "feq": 3 }
      ]
    },
    {
      "name": "Bill",
      "age": 26,
      "hobbies": [
        { "title": "sports", "feq": 5 },
        { "title": "drinking", "feq": 1 }
      ]
    },
    {
      "name": "Mary",
      "age": 26,
      "hobbies": [
        { "title": "cars", "feq": 5 },
        { "title": "drinking", "feq": 1 }
      ]
    }
  ]
}
```

if we need to find all users which has "sport" as one of his hobbies, we can do that:

```js
db.users.find({ "hobbies.title": "sports" });
```

```js
db.users.find({ hobbies: { $size: 3 } });
```

searching for users who have 3 hobby elements in the hobbies array.

```js
db.boxOffice.find({ genre: { $all: ["thriller", "action"] } });
```

searching for movies which's genre contains "thriller" and "action", and the sequence won't matter.

Let's say we have the following data in our db:

```js
[
  {
    name: "John",
    age: 18,
    hobbies: [{ title: "sports", feq: 2 }, { title: "cooking", feq: 3 }]
  },
  {
    name: "Bill",
    age: 26,
    hobbies: [{ title: "sports", feq: 5 }, { title: "drinking", feq: 1 }]
  },
  {
    name: "Mary",
    age: 26,
    hobbies: [{ title: "cars", feq: 5 }, { title: "drinking", feq: 1 }]
  },
  {
    name: "Smith",
    age: 56,
    hobbies: [
      { title: "cars", feq: 5 },
      { title: "drinking", feq: 1 },
      { title: "mtg", feq: 10 }
    ]
  },
  {
    name: "Kevin",
    age: 21,
    hobbies: [{ title: "cars", feq: 5 }, { title: "sports", feq: 1 }]
  }
];
```

we want to search for users who has a sports as one of this hobbies, and the fequency must greater than 3, how can we make our query?

If we do something like

```js
db.users.find({ "hobbies.title": "sports", "hobbies.feq": { $gt: 3 } });
```

This will give us back both "Bill", "Kevin", because this query means user has a sports hobby and user also has a hobbie which frequency is greater than 3

The correct way to do it is

```js
db.users.find({
  hobbies: { $elemMatch: { title: "sports", feq: { $gt: 3 } } }
});
```

### 5.4 Cursor

#### 5.4.1 Basic cursor operations

```js
db.movies.find().count();
```

get the total count for a cursor.

If we have a cursor, we can use the `.next()` method to load the next bunch of data.
Also, we can iterate the cursor by using `.forEach(callback)` to do it. And after it iterated to the end, you can't call `.next()` again, because it's at the end of the documents. To see if you can call the `.next`, you can call `hasNext()`.

#### 5.4.2 Sorting, limit and skip cursor results

```js
var movieCurosr = db.movies.find();
movieCursor.sort({ runtime: 1, "rating.average": -1 });
```

get back the cursor of mathing documents (all documents), and sort them primarily by runtime in asd (1) order and rating.average in desc (-1) order.

Note: this cursor sorting is sorting for all the documents that matching the find query (in here, it's all the documents in the collection), not just each bunch of documents.

```js
db.movies
  .find()
  .sort({ "rating.average": 1 })
  .skip(200);
```

it will take the highest 40 movies cursor.

```js
db.movies
  .find()
  .sort({ "rating.average": 1 })
  .skip(200)
  .limit(10);
```

it will take first 10 movies out of the highest 40 movies.

And without aggregation framework, the order of the `sort`, `limit` and `skip` does not matter, it will get back the same result even we change the order of the chaining, the mongodb will always do it by `sort -> skip -> limit` order.

### 5.5 Projection

Projection specifies which fields to return. Let's say for our app, we only care about the title, the rating and the runtime, all the other fields do not matter.

```js
db.movies.find({}, { name: 1, runtime: 1, "rating.average": 1 });
```

this query will get all the movie documents with only the fields of "name", "runtime", "rating: { average: xxx }" and the \_id. The \_id is always added to the projection, we have to explicitly exclude it by using `_id: 0` to do so.

```js
db.movies.find({ genres: "Drama" }, { name: 1, genres: { $slice: 2 } });
```

this query will get back the name field, and the only two genres in the genre field. And if we do `genres: { $slice: [1, 2] }` instead, it will skip the first genres in the genre array and take two from the second one.

## 6. Update in detail

### 6.1 updateOne, updateMany and \$set

Let's use the userData db, and use the user data set.

Let's say we want to update the hobbies field for Chris, make it like other people's format, we can do this:

```js
db.users.updateOne(
  { _id: ObjectId("5c485da02da99d09ed23b421") },
  {
    $set: {
      hobbies: [
        { title: "Sports", frequency: 5 },
        { title: "Cooking", frenquency: 3 },
        { title: "Hiking", frenquency: 1 }
      ]
    }
  }
);
```

This will set the hobbies array to the array we specified, and keep other fields untouched.

Now if we need to add a field called `isSporty`, and the value is `true` when the person has a Sport as hobby and false otherwise, we can do this:

```js
db.users.updateMany(
  { "hobbies.title": "Sports" },
  { $set: { isSporty: true } }
);
```

In short, the `$set` will modify the existing field value if the field exists, and create the new field if not.

Note that `$set` can update multiple fields at once. For example, we want to update both the name field and the phone field for Chris, we can do this:

```js
db.users.updateOne(
  { _id: ObjectId("5c485da02da99d09ed23b421") },
  { $set: { age: 100, phone: 8888888 } }
);
```

### 6.2 Dealing with number fields

Increase a number:

```js
db.users.updateOne({ name: "Anna" }, { $inc: { age: 2 } });
```

This will increase the age for Anna by 2. And also, the \$inc operator can combine with other operators like usual:

```js
db.users.updateOne(
  { name: "Manuel" },
  { $inc: { age: 2 }, $set: { isSporty: false } }
);
```

Updating Manuel's age and also isSporty fields.

What if we have more complicated senarios? Let's say we want to change Chirs's age to 35 if his age is less than 35. In another word, Chirs's age is maxinum 35. We can do this:

```js
db.users.updateOne({ name: "Chris" }, { $max: { age: 35 } });
```

This does not do anything, because Chirs was originally 100 years old. But if we do the opposite:

```js
db.users.updateOne({ name: "Chris" }, { $min: { age: 35 } });
```

Chris's age becomes 35 now.

### 6.3 Working with fields

Let's say we want to remove a mistyped field on Anna called `isPorty`, we can do this:

```js
db.users.updateOne(
  { name: "Anna" },
  { $unset: { isPorty: "This can be any value" } }
);
```

Actually, if the `isSporty` field is not there, we can simply rename the field by doing this:

```js
db.users.updateOne({ name: "Anna" }, { $rename: { isPorty: "isSporty" } });
```

Be aware, the field value will not change.

### 6.4 Upsert

If we want to update a user called Maria with a document, and we don't really know if Maria is exist in our database, but we want to insert it if she's not in our db, we can do this:

```js
db.users.updateOne(
  { name: "Maria" },
  {
    $set: {
      age: 33
    }
  },
  { upsert: true }
);
```

```js
db.sports.updateOne(
  { title: "basketball" },
  { $set: { requiresTeam: true } },
  { upsert: true }
);
db.sports.update(
  { title: "sumo" },
  { $set: { requiresTeam: false } },
  { upsert: true }
);

db.sports.updateOne({ requiresTeam: true }, { minimumPlayers: 11 });

db.sports.updateOne(
  { minimumPlayers: { $type: "number" } },
  { $inc: { minimumPlayers: 10 } }
);
```

### 6.4 Updating Array Elements

If we want to update the users who has sports as one of his hobbies and he need to do sports more than twice a week how can we do that?

First, let's see how we can find those users:

```js
db.users.find({
  $and: [{ "hobbies.title": "Sports" }, { "hobbies.frequency": { $gt: 2 } }]
});

// or maybe we can also do in this way
db.users.find({ "hobbies.title": "Sports", "hobbies.frequency": { $gt: 2 } });
```

### Unfortunately, this was wrong, it will also give us Anna, her hobbies field is like this:

```json
[{ "title": "Sports", "frequency": 2 }, { "title": "Yoga", "frequency": 3 }]
```

The correct way to get query is using the elemMatch

```js
db.users.find({
  hobbies: { $elemMatch: { title: "Sports", frequency: { $gt: 2 } } }
});
```

This time, Anna is not included in our results anymore.

Now, if we want to update those Sports hobbies which has more than twice a week by appending a new field called "highFrequency" and set its value to true, we can do this:

```js
db.users.updateMany(
  {
    hobbies: { $elemMatch: { title: "Sports", frequency: { $gt: 2 } } }
  },
  { $set: { "hobbies.$.highFrequency": true } }
);
```

Let's do another example, here we want to update the Cooking hobbies which has more than 4 times a week by increase its value by 1

```js
db.users.updateMany(
  {
    hobbies: { $elemMatch: { title: "Cooking", frequency: { $gt: 4 } } }
  },
  { $inc: { "hobbies.$.frequency": 1 } }
);
```

Here the "\$" means the matched element in the array.

If this time, instead of updating the matched document, we want to reduce all hobbies frequency by 1 if the user is more than 29 years old.

```js
db.users.updateMany(
  { age: { $gt: 29 } },
  { $inc: { "hobbies.$[].frequency": -1 } }
);
```

The `$[]` means all the documents under the hobbies field.

Let's change our requirement a bit, this time, we want to update all the hobbies sub documents if the frequency is greater than 2, we want to append a "goodFrequency" field and set it to true, how can we achieve that?

```js
db.users.updateMany(
  { "hobbies.frequency": { $gt: 2 } },
  { $set: { "hobbies.$[el].goodFrequency": true } },
  { arrayFilters: [{ "el.frequency": { $gt: 2 } }] }
);
```

here the `el` is something we defined, it means all the sub documents in the hobbies array that matches the arrayFilters. Notice here, the arrayFilters is an array of filter documents, it can be multiple of them. Also, the filter on the top can be totally different than the filter on the bottom.

This time, we want to add a hobby which is `{title: "Mtg", frequency: 2}` to the max's hobbies, instead of rewrite all the hobbies by using `$set`, we can use another way to do it

```js
db.users.updateOne(
  { name: "Max" },
  { $push: { hobbies: { title: "Mtg", frequency: 2 } } }
);
```

the `$push` only working for array fields.

Actually, there are more options you can do with \$push,

```js
db.users.updateOne(
  { name: "Maria" },
  {
    $push: {
      hobbies: {
        $each: [
          // 1
          { title: "Basketball", frequency: 2 },
          { title: "Shopping", frequency: 2 }
        ],
        $sort: { frequency: -1 }, // 2
        $slice: 2 // 3
      }
    }
  }
);
```

- 1: if you want to push multiple things into the array, you can do that by \$each

- 2: `$sort` means you after your inserting the elements to the array, it will re-order the array as request, 1 means asd -1 means desc.

- 3: `$slice` means you want to leave x things after the insert, note here if your array has more elements than the \$slice, it will remove elements instead of insert.

If you only want to add one element to the array, there is another way to do that:

```js
db.users.updateOne(
  { name: "Max" },
  { $addToSet: { hobbies: { title: "Mtg", frequency: 2 } } }
);
```

This will give the same result, as the `$push` does. The difference is, `$push` allows you to push multiple same documents, and `$addToSet` will not do anything if the document is already there.

Now, this time, we want to remove some documents from Maria's hobbies, if we want to remove all the `Shopping`s, we can do this:

```js
db.users.updateOne(
  { name: "Maria" },
  { $pull: { hobbies: { title: "Shopping" } } }
);
```

Here, the `$pull` operator will remove all the sub documents which meets the criteria, and the criteria here does not have to be equality, you can also have something like `frequency: {$gt: 3}`

And if you simply want to remove the first or the last n items, you can use `$pop`

```js
db.users.updateOne(
  {
    name: "Max"
  },
  { $pop: { hobbies: 2 } }
);
```

This will remove the last 2 elements from the Max's hobbies. If you specify a `-2` it will remove the first elements.

## 7. Delet Documents

```js
db.users.deleteOne({ _id: ObjectId("5c4b0fd4799b842e752dc000") });
```

Normally, if we know the id of the document we want to delete, just use the \_id, because it will not accidentally delete other documents.

```js
db.users.deleteMany({ age: { $gt: 30 }, isSporty: true });
```

this will delete all the documents which age is greater than 30 and also a sporty.

Also, we can delete all the documents in a collection

```js
db.users.deleteMany({});
```

Or we can delete the entire collection

```js
db.users.drop();
```

Note here this will normally not working in most of the drivers.

Even we can delete the database by

```js
db.dropDatabase();
```

This will drop the database you currently working on.

## 8. Index

Index is used for making query faster, it added to one or many fields. But index come with a cost, when we insert or update, we need to modify the indexes too, so don't add index on all the fields, only added if necessary.

Let's import the persons.json to our database, here if we want to get all contacts who is more than 60 years old, we can do this:

```js
db.contacts.find({ "dob.age": { $gt: 60 } });
```

We can use addtional chain to analysis our query:

```js
db.contacts.explain("executionStats").find({ "dob.age": { $gt: 60 } });
```

Then we got something like this:

```json
{
  "queryPlanner": {
    "plannerVersion": 1,
    "namespace": "contactData.contacts",
    "indexFilterSet": false,
    "parsedQuery": {
      "dob.age": {
        "$gt": 60
      }
    },
    "winningPlan": {
      "stage": "COLLSCAN",
      "filter": {
        "dob.age": {
          "$gt": 60
        }
      },
      "direction": "forward"
    },
    "rejectedPlans": []
  },
  "executionStats": {
    "executionSuccess": true,
    "nReturned": 1222,
    "executionTimeMillis": 5,
    "totalKeysExamined": 0,
    "totalDocsExamined": 5000,
    "executionStages": {
      "stage": "COLLSCAN",
      "filter": {
        "dob.age": {
          "$gt": 60
        }
      },
      "nReturned": 1222,
      "executionTimeMillisEstimate": 0,
      "works": 5002,
      "advanced": 1222,
      "needTime": 3779,
      "needYield": 0,
      "saveState": 39,
      "restoreState": 39,
      "isEOF": 1,
      "invalidates": 0,
      "direction": "forward",
      "docsExamined": 5000
    }
  },
  "serverInfo": {
    "host": "ZhengdeMBP",
    "port": 27017,
    "version": "4.0.3",
    "gitVersion": "7ea530946fa7880364d88c8d8b6026bbc9ffa48c"
  },
  "ok": 1
}
```

In this output, we can see a couple of things:

- 1. `winningPlan`: this is how the query gets executed, which means it scans each collumns.
- 2. `executionStats -> executionTimeMillis`: this means it takes 5 miliseconds to execute the query.
- 3. `executionStats -> totalDocsExamined`: the query scans in total of 5000 documents.

Here we can see, the query is not very effecient, because it scans 5000 documents to get only 1222 results back.

Now let's try to add some indexes and see how it gets changed.

To add an index to a collection, we can do this:

```js
db.contacts.createIndex({ "dob.age": 1 });
```

We can add index to nested fields just like normal fields, and the value `1` means sort the documents by asd order (-1 means desc order).

And now, let's execute the last command again too see the analys result:

```json
{
  "queryPlanner": {
    "plannerVersion": 1,
    "namespace": "contactData.contacts",
    "indexFilterSet": false,
    "parsedQuery": {
      "dob.age": {
        "$gt": 60
      }
    },
    "winningPlan": {
      "stage": "FETCH",
      "inputStage": {
        "stage": "IXSCAN",
        "keyPattern": {
          "dob.age": 1
        },
        "indexName": "dob.age_1",
        "isMultiKey": false,
        "multiKeyPaths": {
          "dob.age": []
        },
        "isUnique": false,
        "isSparse": false,
        "isPartial": false,
        "indexVersion": 2,
        "direction": "forward",
        "indexBounds": {
          "dob.age": ["(60.0, inf.0]"]
        }
      }
    },
    "rejectedPlans": []
  },
  "executionStats": {
    "executionSuccess": true,
    "nReturned": 1222,
    "executionTimeMillis": 3,
    "totalKeysExamined": 1222,
    "totalDocsExamined": 1222,
    "executionStages": {
      "stage": "FETCH",
      "nReturned": 1222,
      "executionTimeMillisEstimate": 0,
      "works": 1223,
      "advanced": 1222,
      "needTime": 0,
      "needYield": 0,
      "saveState": 9,
      "restoreState": 9,
      "isEOF": 1,
      "invalidates": 0,
      "docsExamined": 1222,
      "alreadyHasObj": 0,
      "inputStage": {
        "stage": "IXSCAN",
        "nReturned": 1222,
        "executionTimeMillisEstimate": 0,
        "works": 1223,
        "advanced": 1222,
        "needTime": 0,
        "needYield": 0,
        "saveState": 9,
        "restoreState": 9,
        "isEOF": 1,
        "invalidates": 0,
        "keyPattern": {
          "dob.age": 1
        },
        "indexName": "dob.age_1",
        "isMultiKey": false,
        "multiKeyPaths": {
          "dob.age": []
        },
        "isUnique": false,
        "isSparse": false,
        "isPartial": false,
        "indexVersion": 2,
        "direction": "forward",
        "indexBounds": {
          "dob.age": ["(60.0, inf.0]"]
        },
        "keysExamined": 1222,
        "seeks": 1,
        "dupsTested": 0,
        "dupsDropped": 0,
        "seenInvalidated": 0
      }
    }
  },
  "serverInfo": {
    "host": "ZhengdeMBP",
    "port": 27017,
    "version": "4.0.3",
    "gitVersion": "7ea530946fa7880364d88c8d8b6026bbc9ffa48c"
  },
  "ok": 1
}
```

### 7.1 How index works?

Here, we can see a couple of changes:

- 1. `executionTimeMillis`: is down to 3 miliseconds.
- 2. `totalDocsExamined`: is down to 1222 documents.
- 3. `winningPlan`: is `IXSCAN` and split into two steps.

In detail, you can think index a pointer which pointing to the memory of the document, and it is sorted.

Let's say we have some person data:

```
[
  {name: "jone", age: 38, hobbies: ... },
  {name: "billy", age: 16, hobbies: ... },
  {name: "anna", age: 22, hobbies: ... },
  {name: "john", age: 14, hobbies: ... },
  {name: "pete", age: 27, hobbies: ... },
]
```

if we try to query the data set by age, it has to scan all the documents.

the `db.contacts.createIndex({ "dob.age": 1 })` will create something like this:

```
(14, <Memory 3>),
(16, <Memory 0>),
(22, <Memory 2>),
(27, <Memory 4>),
(38, <Memory 1>)
```

if we query it like

```js
db.persons.find({ age: { $lt: 20 } });
```

it will scan to the third memory pointer, and sees it already 22, so it will stop and only fetches the first two documents accroding to their Memory address in the computer.

Now let's modify our query a bit:

```js
db.contacts.explain("executionStats").find({ "dob.age": { $gt: 0 } });
```

The result is somehow suprise you, we can see that the excution time is going up to 9 miliseconds, why?

Because it returns all the documents so it simply go through the entire index collection and it also need to fetch the data according to the memory the indexes pointing to.

Let's remove our index by

```js
db.contacts.dropIndex({ "dob.age": 1 });
```

Now the execution time is back to 5 miliseconds.

_In conclusion, if the query will return all or most of the documents in the collection, use index will even make the execution slower._

### 7.2 compond index

If we want to query the users who are male, shall we use index?

```js
db.contacts.find({ gender: "male" });
```

The answer is "No" because we knew that people's gener will either be male or female, so querying males will return about half of the documents. After we try it, the execution will take 3 miliseconds which is not very significantly speed up.

However, if we want to query users who are male and older than 60 years old, we can use some compond index.

```js
db.contacts.createIndex({ "dob.age": 1, gender: 1 });
```

Now the execution time is shorten from 5 miliseconds to 3 miliseconds which is a significant speed up.

Notes here, the sequence of the keys matters, this is like sort by age then gender.

The compond index is not creating two indexes, it creating only one like this

```
(30 male, <Memory 3>),
(30 male, <Memory 20>),
(31 male, <Memory 29>),
(31 female, <Memory 37>),
...
```

Also, if we query only on age, which is the first key of the index, it will also get benefit from our index, but if we want to query the gender, which is not the first key of the index, it will still use the `COLLSCAN` rather than the `IXSCAN`.

Index is not only work on querying, it also works on sorting

```js
db.contacts.find({ "dob.age": 35 }).sort({ gender: 1 });
```

This will also speed up if we have the previous index.

And to see what indexes are there in the collection, we can do

```js
db.contacts.getIndexes();
```

Note that there is always a default index on the \_id of the documents.

### 7.3 Unique index

Sometime, we want our data has a unique field such as email or username, for this usecase, we can also create an index with some configurations:

```js
db.contacts.createIndex({ email: 1 }, { unique: true });
```

It will give error if we have some duplicate emails, and each time, we insert a person, mongodb will ensure the email is unique, otherwise it will also give us error.

### 7.4 Partial filter

Let's say, we are building an application, that need always query male and age, for instance:

```js
db.contacts.find({ "dob.age": { $gt: 70 }, gender: "male" });
```

then we can create a partial index

```js
db.contacts.createIndex(
  { "dob.age": 1 },
  { partialFilterExpression: { gender: "male" } }
);
```

This is a partial index, it only stores the pointers for male and sorted by age, females are not stored in our indexes. But why we do that?

Because if we knew that we need to query the gender of males and find the age very often, using this partial index can save us some memory in our disk, and also it will speed up the insert and update for the collection since the index collection is smaller than the full index.

However, if we query more than our index such as only query about the age, the mongodb will still use the "COLLSCAN" rather than the "IXSCAN" because the engine do not want us to lose some data.

### 7.5 Time-to-live index

Let's say we have a collection of sessions data which is inserted like this:

```js
db.sessions.insertOne({ data: "asdadads", createdAt: new Date() });
```

Now if we want each documents get deleted by itself automatically after 10 seconds, we can use the index to do that:

```js
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 10 });
```

This will delete the index after 10 second it added. But remember, if we have some data before we createIndex, it will also gets evaluated after we insert a new document since each time a insertion, update and deletion will re-index the collection. And this `expireAfterSeconds` only works for Date data.

### 7.6 Query Diagnosis and Query planning

When we chaining the `explain` operator on our query, we can pass the following arguments into it:

- "queryPlanner": this will give us the Summary for executed query + winning plan

- "executionStats": this will give us the Summary for the executed query + winning plan + possibly rejected plans

- "allPlansExecution": this will give us the Summary for the executed query + winning plan + winning plan decision process

This will help us to make better decision about how to create the index against the queries.

#### 7.6.1 Covered queries

To analysis the efficiency of an indexing stretegy, there are several things to be considered:

- Milliseconds of precess time
- The IXSCAN should always beats the COLLSCAN
- \# of keys (in index) examined should be close to the \# of documents examined
- \# of documents examined should be close to the \# of documents returned

If a query only returns the fields in the index (by using a projection normally), this is called covered queries, in other words, the query is covered by the index. This kind of covered queries, if exists in our usecases, can be very fast because it doesn't need to pull the data again by using the pointer since all the related data is in the pointer.

Let's see an example, here we have a data set like this:

```
[
  { name: "foo", salary: 5000, age: 25 },
  { name: "bar", salary: 6000, age: 35 },
  { name: "baz", salary: 8000, age: 29 },
```

if we only care about the name, like

```js
db.customers.find({ name: "foo" }, { _id: 0, name: 1 });
```

we can create an index on name

```js
db.customers.createIndex({ name: 1 });
```

if we check the stats by chaining the `.explain("executionStats")`, we can see the `totalDocsExamined` is 0, because it simply doesn't have to examine any documents, the index covers all the needs.

this covered query is also working on compond indexes, for instance, if we need to query all the person named "foo" and also the salary more than 1000 (`db.customers.find({name: "foo", salary: {$gt: 1000}}, {_id: 0, name: 1, salary: 1})`), the previous index is no longer makes a covered query. And if we do this

```js
db.customers.createIndex({ name: 1, salary: 1 });
```

This index will again make our query a covered query.

#### 7.6.2 Reject Plan

Let's added two indexes into our customer data set

```js
db.customers.createIndex({ name: 1 });

db.customers.createIndex({ age: 1, name: 1 });
```

Now let's query for name and age and try to see the explainations:

```js
db.customers.explain().find({ name: "Max", age: 30 });
```

```
"winningPlan" : {
  "stage" : "FETCH",
  "inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"age" : 1,
					"name" : 1
				},
				"indexName" : "age_1_name_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"age" : [ ],
					"name" : [ ]
				},
      }
      ...
      "rejectedPlans" : [
			{
				"stage" : "FETCH",
				"filter" : {
					"age" : {
						"$eq" : 30
					}
				},
				"inputStage" : {
					"stage" : "IXSCAN",
					"keyPattern" : {
						"name" : 1
					},
}
```

Here mongodb used our compound index instead of the name index, because somehow it is more efficient. But how does mongodb come up this solution?

In mongodb, when we query some data and there are some indexes, mongodb will try to query the first 1000 data by using each indexes and compare the time of execution, then pick up the fastest index. Also, mongodb will cache the winning plan for this exact query (field and name), for a while, so next time, if we query the same thing, mongodb will automatically use the last winning plan.

There are some reasons that mongodb will clear out the cache:

- Write threshhold (1000)
- Index rebuild
- Other indexes are added or removed
- Mongodb server restarted

### 7.7 Multi-key index

Let's create a new users collection by inserting the data like this:

```js
db.users.insertOne({
  name: "Max",
  hobbies: ["Cooking", "Sports"],
  addresses: [{ street: "Main street" }, { street: "Second street" }]
});
```

And let's add an index on hobbies

```js
db.users.createIndex({ hobbies: 1 });
```

And query for hobbies contains "Sports"

```js
db.users.explain("executionStats").find({ hobbies: "Sports" });
```

Here if you read the output carefully, you can see the winning plan is our index, and there is a field called `isMultiKey` is true this time. This means our key is a multiKey index. And it works just like the single key or compound key index, but you need to be careful, if you have an array contains about 4 items and you have 1000 documents, the key will be 4 X 1000 of its size, so unless you need to query that field very often, do not use it.

Let's see another example, let's add another key on the addresses field:

```js
db.users.createIndex({ addresses: 1 });
```

and let's query against the street of the address

```js
db.users.explain("executionStats").find({ "addresses.street": "Main street" });
```

Now you can see the output, the winning plan is `COLLSCAN`, this is because the index can not reach to the inside of each object of the field, so if we do this:

```js
db.users
  .explain("executionStats")
  .find({ addresses: { street: "Main street" } });
```

it will use the `IXSCAN` plan.

So if we often query the street like `find({ "addresses.street": "Main street" })`, we can define our index like this:

```js
db.users.createIndex({ "addresses.street": 1 });
```

This will create the index against `addresses.street` field, and the result is using the `IXSCAN`.

Note here, the multiKey index can be a part of a compound index, like this:

```js
db.users.createIndex({ name: 1, hobbies: 1 });
```

but you can't have compound indexes which has more than 1 multiKey index like this:

```js
db.users.createIndex({ addresses: 1, hobbies: 1 });
```

This is not working, and the reason is simple, it will create too many index records.

### 7.8 Text index

When we search for text, we normally won't use regex, because it has very poor performance, instead, we create some text indexes, which only cover the keywords, not the stop words (this, that, and, a, the...). Text index works like multiKey index.

Let's create some product data:

```js
db.products.insertMany([
  {
    title: "A book",
    description: "This is an awesome book to get some understanding of Mongodb"
  },
  {
    title: "A T-shirt",
    description: "This T-shirt is red, and it's awesome"
  }
]);
```

Now let's create a text index on our description field

```js
db.products.createIndex({ description: "text" });
```

Here, instead of using `1` or `-1`, we use `text`, which is a special value in mongodb to create text indexes.

To search the text, we normally don't use `{field: <Text>}`, because this is not a text search, it search for equality. We use special operators to query the text:

```js
db.products.find({ $text: { $search: "awesome" } });
```

Note, this `$text` will only work if we have a text index, and it will only search for the field with the text index. Also, you can only have ONE text index per collection.

This will search the word `awesome` for all the fields of our documents. And this is much more faster than regex. And text indexes are case insensitive, "AWESOME", "Awesome" and "awesome" will give you the same out comes.

If you want to search some terms (more than one word combination), you have to use this:

```
db.products.find({ $text: { $search: "\"awesome book\"" } });
```

If you search like this:

```js
db.products.find({ $text: { $search: "awesome book" } });
```

it will return you all the documents either contains "awesome" or "book".

We can also see the score of the text search, and we can sort the search result base on the score:

```js
db.products.find(
  { $text: { $search: "awesome book" } },
  { score: { $meta: "textScore" } }
);
```

And now the result is like this

```js
{ "_id" : ObjectId("5c4d406e054ef8b656e9116f"), "title" : "A book", "description" : "This is an awesome book to get some understanding of Mongodb", "score" : 1.2 }
{ "_id" : ObjectId("5c4d406e054ef8b656e91170"), "title" : "A T-shirt", "description" : "This T-shirt is red, and it's awesome", "score" : 0.625 }
```

the higher score means more relevent to the search keywords

to sorting against the score, we can do this:

```js
db.products
  .find(
    { $text: { $search: "awesome book" } },
    { score: { $meta: "textScore" } }
  )
  .sort({ score: { $meta: "textScore" } });
```

This will sort the result on its score, notes here, you need the projection for score then you can sort against it.

Now, we knew that we can only have one index per collection, but what if we want more than one indexes? Like here, what if we want to search for both title and descriptions?

Now let's insert another product into our products collection

```js
db.products.insertOne({
  title: "Red Bull",
  description: "An awesome drink can make you feel good"
});
```

and let's query the text in the title:

```js
db.products.find({ $text: { $search: "Bull" } });
```

We see nothing is returned, this is because the text index is only on the description field, and for all the three documents, there is no "Bull" in their descriptions. Now let's create another index.

First let's remove the index by using some special ways:

```js
// get the name of the index
db.products.getIndexes(); // name is "description_text"

// drop the index
db.products.dropIndex("description_text");
```

Then we add a combined text index:

```js
db.products.createIndex({ title: "text", description: "text" });
```

And this time, we run the query `find({ $text: { $search: "Bull" } })` again, we can see the documents with title of "Red Bull" returned.

Also, we can exclude words in text search by simply putting the `-` sign before the text. Now, if we only want to search for "awesome" but we don't want to include "t-shirt" we can do this:

```js
db.products.find({ $text: { $search: "awesome -t-shirt" } });
```

### 7.9 Buiding index

If we have a large collection, building the index will take time, especially for the complicated ones such as multiKey and text indexes, by default, during the time of creating the index, the database is locked, so you can't do anything until the building process completed.

Overall, there are two ways to build indexes:

- Foreground: building index as a synchorized task, the db is locked until the building process completed. And this is faster for building process.

- Background: building index as an asynchorized task, although it is much more slower than the Foreground, the database is not locked during the process.

So in general, in the production database, you better use the Background build for indexes, because it will not block other tasks.

By default, the createIndex is using the foreground way to build the index, to ulter it, we can do this:

```js
db.users.createIndex({ name: 1 }, { background: true });
```

## 8. Aggregation framework

Aggregation framework is just like the `find` method, it give you some steps to get the data you needed.

Aggregation framework works as follow:

collection -> {$match} -> {$sort} -> {$group} -> {$project} -> output (a list of documents)

All aggregation framework about is transform the data you need through a pipline of steps.

Let's use our contactData database.

### 8.1 Getting start

To start a aggregation, instead of `.find`, we use `.aggregate([])` and inside the array, there are some steps, each step tells the mongodb how to transform the data. Also, aggregation framework can also take advantage from things like indexes.

```js
db.contacts.aggregate([{ $match: { gender: "female" } }]);
```

This will return exactly like

```js
db.contacts.find({ gender: "female" });
```

### 8.2 Group

```js
db.contacts.aggregate([
  { $match: { gender: "female" } },
  {
    $group: {
      _id: { livingState: "$location.state" },
      totalPersonCount: { $sum: 1 }
    }
  }
]);
```
This will return you something like this:
```js
{ "_id" : { "livingState" : "tayside" }, "totalPersonCount" : 1 }
{ "_id" : { "livingState" : "berkshire" }, "totalPersonCount" : 1 }
{ "_id" : { "livingState" : "michigan" }, "totalPersonCount" : 1 }
{ "_id" : { "livingState" : "county down" }, "totalPersonCount" : 1 }
{ "_id" : { "livingState" : "cornwall" }, "totalPersonCount" : 2 }
```
What does it do?
It groups our data by state, and count the number of people with that state.

Each group need to have an _id, here, in group, it should be a document. The $location.state means to get the value of location.state. the $sum is a special operator which will add the value of the $sum each time we found a record.

We can also sort the data after grouping by totalPersonCount
```js
db.contacts.aggregate([
  { $match: { gender: "male" } },
  {
    $group: {
      _id: { livingState: "$location.state" },
      totalPersonCount: { $sum: 1 }
    }
  },
  {$sort: { totalPersonCount: -1}}
]);
```
The data is sorted after the group step, which is something that simple query can never do.

Let's do another aggregation for practise, this time, let's find all the persons who is older than 50 years old, and group them by gender to display the number of people and the average age for each gender, and then group them by the total number per gender.

```js
db.contacts.aggregate([
  { $match: { "dob.age": {$gt: 50 }}},
  { 
    $group: {
      _id: { gen: "$gender" },
      totalNumber: {$sum: 1},
      averageAge: { $avg: "$dob.age"}
    }
  },
  { $sort: {totalNumber: -1}}
])
```

### 8.3 Project
Projection in the aggregation framework is much more powerful than it in the normal query.

Let's say we want to get a new field "fullName" which is the first name concatenate the last name, how can we do that?

```js
db.contacts.aggregate([
  {$project: {
    _id: 0, gender: 1,
    fullName: {$concat: ["$name.first", " ", "$name.last"]}
  }}
])
```
Let's make it more difficult, say we want the name to be title case, for example, "Jasen Pan".

```js
db.contacts.aggregate([
  {$project: {
    _id: 0, gender: 1,
    fullName: {
      $concat: [
        {$toUpper: { $substrCP: ["$name.first", 0, 1]}},
        {
  $substrCP: ["$name.first", 1, { $subtract: [{ $strLenCP: "$name.first" }, 1] }]
},
        " ",
        {$toUpper: { $substrCP: ["$name.last", 0, 1]}},
        {
  $substrCP: ["$name.last", 1, { $subtract: [{ $strLenCP: "$name.last" }, 1] }]
}
      ]
    } 
  }}
])
```
<!-- That's better, but some of the first names or last names has more than one word, we also need to address that

```js
db.contacts.aggregate([
  {$project: {
    _id: 0, gender: 1,
    fullName: {
      $concat: [
        {$toUpper: { $substrCP: ["$name.first", 0, 1]}},
        {
  $substrCP: ["$name.first", 1, { $subtract: [{ $strLenCP: "$name.first" }, 1] }]
},
        " ",
        {$toUpper: { $substrCP: ["$name.last", 0, 1]}},
        {
  $substrCP: ["$name.last", 1, { $subtract: [{ $strLenCP: "$name.last" }, 1] }]
}
      ]
    } 
  }}
])
``` -->

Note here, each step in a pipeline can be repeated, for example, you can have something like 
```js
db.contacts.aggregate([
  {$project: {...}},
  {$group: {...}},
  {$project: {...}},
  {$group: {...}},
  {$sort: {...}}
])
```
This is totally fine, just remember, the output from the last step will be the input for next step in the pipeline.

Let's do a practise, now, beside the fullName and the gender, we also want the email, and another field called location, which should be a geoJson array, look like this:
```js
location: {
  type: "Point",
  coordinates: [
    <longitude>, // has to be a number
    <latitude>  // has to be a number
  ]
}
```

```js
db.contacts.aggregate([
  {
    $project: {
      _id: 0,
      gender: 1,
      email: 1,
      name: 1,
      location: {
        type: "Point",
        coordinates: [
          {
            $convert: {
              input: "$location.coordinates.longitude",
              to: "double",
              onError: 0.0,
              onNull: 0.0
            }
          },
          {
            $convert: {
              input: "$location.coordinates.latitude",
              to: "double",
              onError: 0.0,
              onNull: 0.0
            }
          }
        ]
      }
    }
  },
  {
    $project: {
      gender: 1,
      email: 1,
      fullName: {
        $concat: [
          { $toUpper: { $substrCP: ["$name.first", 0, 1] } },
          {
            $substrCP: [
              "$name.first",
              1,
              { $subtract: [{ $strLenCP: "$name.first" }, 1] }
            ]
          },
          " ",
          { $toUpper: { $substrCP: ["$name.last", 0, 1] } },
          {
            $substrCP: [
              "$name.last",
              1,
              { $subtract: [{ $strLenCP: "$name.last" }, 1] }
            ]
          }
        ]
      },
      location: 1
    }
  }
]);
```
Now, let's also try to convert the birthday to the top level field as well as the age.

```js
{
    $project: {
      _id: 0,
      gender: 1,
      email: 1,
      name: 1,
      birthdate: {
        $dateFromString: {
          dateString: "$dob.date"
        }
      },
      age: "$dob.age",
      location: {
        ...
      }
      ...
```

Also, if we don't need to specify the onError and onNull on the convert step, there are few short cut operator for us such as `$toDate` or `$toDouble`, now we can say `birthday: { $toDate: "$dob.date"}`, it will give us the same result.

Let's do another level of transformation, let's group the documents by the birthday year, and count how many people born on each year:
```js
db.contacts.aggregate([
  {
    // projection level
  },
  { $group: { _id: { birthYear: { $isoWeekYear: "$birthdate" } }, numPersons: { $sum: 1 } } },
    { $sort: { numPersons: -1 } }
]);

```

### 8.4 working with arrays
Now let's do some stuff on "friends.json" data set. The first requirement is we want to get a summary which shows each age as a group and the names for that age

```js
db.friends.aggregate([{$group: {_id: {age: "$age"}, names: {$push: "$name"}}}]
```
This is simple, now what if we want to get all the hobbies contained for each age?
If we use the previous aggregation steps, it won't work, because it will simply add the array to the array.

Here we use the `$unwind` operator, this operator break the array field into multiple documents, example:

```js
// persons
[
  {name: "Foo", hobbies: ["play", "eat"]},
  {name: "Bar", hobbies: ["drink", "smoke"]}
]

db.persons.aggregate([
  {$unwind: "$hobbies"}
])

// result
[
  {name: "Foo", hobbies: "play"},
  {name: "Foo", hobbies: "eat"},
  {name: "Bar", hobbies: "drink"},
  {name: "Bar", hobbies: "smoke"}
]
```

Now let's solve our problem:
```js
db.friends.aggregate([
  { $unwind: "$hobbies" },
  {
    $group: {
      _id: { age: "$age" },
      allHobbies: {
        $push: "$hobbies"
      }
    }
  }
]);
```
We firstly break the documents down according toe the hobbies array, then we group these documents back by age. But there is a problem, the "Cooking" appeared twice.

To eliminate the duplicated values, we can use `$addToSet` rather than `$push`

Now if we want to get only the first score for the exams, we can use the `$slice` operator in the projection.

```js
// get back the first score
db.friends.aggregate([
  {$project: { _id: 0, scores: {$slice: ["$examScores", 1]}}}
])

// get back the last score
db.friends.aggregate([
  {$project: { _id: 0, scores: {$slice: ["$examScores", -1]}}}
])

// get back the last two scores
db.friends.aggregate([
  {$project: { _id: 0, scores: {$slice: ["$examScores", 1, 2]}}}
])
```

Also, if we only interested in how many exams a friend take, we can use this
```js
db.friends.aggregate([
  { $project: {_id: 0, numOfExams: {$size: "$examScores"}}}
])
```

This time, we only care about the exams which got a score more than 60, here we can use `$filter`, it works similar like the `filter` method in javascript
```js
db.friends.aggregate([
  {
    $project: {
      _id: 0,
      name: 1,
      goodScores: { $filter: {
        input: "$examScores", // 1
        as: "s", // 2
                    // 4
        cond: { $gt: ["$$s.score", 60]} // 3
      }}
    }
  }
])
```
- 1. the field name we want to filter
- 2. the temp variable we want to call
- 3. the condition
- 4. if we want to reference our temp variable, we need to use `$$`

If we want to fetch each student with the highes score, and sort the documents by scores in desending order, we can do this:
```js
db.friends.aggregate([
  {$unwind: "$examScores"},
  {$sort: {"examScores": -1}},
  {$project: {
    name: 1,
    age: 1,
    myScore: "$examScores.score"
  }},
  {$group: {
    _id: "$_id",
    name: {$first: "$name"},
    age: {$first: "$age"},
    highestScore: { $max: "$myScore"}
  }},
  {$sort: {"highestScore": -1}}
])
```
