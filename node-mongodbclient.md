# Creating a MongoDB client using Node.js

[MongoDB Node Documentation](https://www.mongodb.com/docs/drivers/node/current/)

## Setup

```bash
    npm init -y
```

Install the MongoDB package.

```bash
    npm install mongodb
```

## Files

Create a file for the MongoDB connection named ``atlas_uri.js``.

Add your connection string.

```bash
module.exports = { 
    uri: "mongodb://alan:test123@localhost:27018/?retryWrites=true&w=majority&authSource=admin" 
}
```

Create the main entrypoint for our application named ``app.js``. Use this code to test the connection string.

```bash
    const { MongoClient } = require('mongodb');
    const { uri } = require('./atlas_uri');

    console.log(uri);
```

Returns.

> mongodb://alan:test123@localhost:27018/?retryWrites=true&w=majority&authSource=admin

Now search for a list of databases.

```bash
const { MongoClient } = require('mongodb');
const { uri } = require('./atlas_uri');

const client = new MongoClient(uri);
const dbname = 'bank';

const connectToDatabase = async () => {
    try {
        await client.connect();
        console.log(`Connected to the ${dbname} database...`);

    } catch (error) {
        console.error(`Error connecting to the ${dbname} database...`);
    }
};

const main = async () => {
    try {
        await connectToDatabase();
        const dbList = await client.db().admin().listDatabases();
        dbList.databases.forEach(db => console.log(` - ${db.name}`));
    } catch (error) {
        console.error(`Error connecting to the ${dbname} database: ${error}`);
    } finally {
        await client.close();
    }
};

main();
```

Returns.

> Connected to the bank database...
>
> - admin
> - bookAPI
> - config
> - family
> - flights
> - local
> - record-db

### network issues

This code works fie on a local connection but when you are working in MongoDB Atlas Cloud you will have to add your IP address into the **Network Access** list.

## MongoDB CRUD operations in Node.js

### insertOne()

Create a new file, ``insert.js`` to insert a document into a collection.

#### insert.js

```bash
const { MongoClient } = require('mongodb');
const { uri } = require('./atlas_uri');

const client = new MongoClient(uri);
const dbname = 'bank';

const account = {
    account_id: "MDB999569384",
    account_holder: "James Robson",
    account_type: "checking",
    balance: 2956.89,
    transfers_complete: [
        "TR16523347",
        "TR16523346",
        "TR16523345",
        "TR16523344",
        "TR16523343",
    ],
    address: {
        city: "PineTree",
        postcode: 3446,
        street: "Jones St",
        number: 12
    },
};

const main = async () => {
    try {
        await client.connect();

        const database = client.db(dbname);
        const accounts = database.collection('accounts');
        const result = await accounts.insertOne(account);
        console.log(result);
    } catch (error) {
        console.error(`Error connecting to the ${dbname} database: ${error}`);
    } finally {
        await client.close();
    }
};

main();
```

This inserts one document into the database.

```bash
    [{
        _id: ObjectId('65f27156a4c3bfb4d9583b79'),
        account_id: 'MDB999569384',
        account_holder: 'James Robson',
        account_type: 'checking',
        balance: 2956.89,
        transfers_complete: [
          'TR16523347',
          'TR16523346',
          'TR16523345',
          'TR16523344',
          'TR16523343'
        ],
        address: {
          city: 'PineTree',
          postcode: 3446,
          street: 'Jones St',
          number: 12
        }
      }
    ]
```

#### insertMany.js

```bash
const { MongoClient } = require('mongodb');
const { uri } = require('./atlas_uri');

const client = new MongoClient(uri);
const dbname = 'bank';
const collectionName = 'accounts';

const accountsCollection = client.db(dbname).collection(collectionName);

const sampleAccounts = [
    {
        account_id: "MDB999566859",
        account_holder: "Bert Robson",
        account_type: "checking",
        balance: 2256.89,
        last_updated: new Date(),
    },
    {
        account_id: "MDB999566369",
        account_holder: "Giovanny Robson",
        account_type: "checking",
        balance: 6556.89,
        last_updated: new Date(),
    },
]

const main = async () => {
    try {
        await client.connect();

        const result = await accountsCollection.insertMany(sampleAccounts);
        console.log(`Inserted ${result.insertedCount} documents.`);
    } catch (error) {
        console.error(`Error connecting to the ${dbname} database: ${error}`);
    } finally {
        await client.close();
    }
};

main();
```

### findOne()

#### find01.js

```bash
const { MongoClient } = require('mongodb');
const { uri } = require('./atlas_uri');

const client = new MongoClient(uri);
const dbname = 'bank';
const collectionName = 'accounts';

const accountsCollection = client.db(dbname).collection(collectionName);

const query = { account_holder: "Alan Robson" };

const options = {

    // Sort matched documents in descending order by rating
    sort: { "account_holder": -1 },

    projection: { _id: 0, account_id: 1, account_holder: 1 },
};

const main = async () => {
    try {
        await client.connect();

        const account = await accountsCollection.findOne(query, options);
        console.log(account);
    } catch (error) {
        console.error(`Error connecting to the ${dbname} database: ${error}`);
    } finally {
        await client.close();
    }
};

main();
```

Returns:

> { account_id: 'MDB999569387', account_holder: 'Alan Robson' }

### find()

Find a number of documents bases on a regular expression query.

#### findMany.js

```bash
const { MongoClient } = require('mongodb');
const { uri } = require('./atlas_uri');

const client = new MongoClient(uri);
const dbname = 'bank';
const collectionName = 'accounts';

const accountsCollection = client.db(dbname).collection(collectionName);

const query = { account_holder: { "$regex": /Robson/ } };

const options = {

    // Sort matched documents in descending order by rating
    sort: { "account_holder": -1 },

    projection: { _id: 0, account_id: 1, account_holder: 1 },
};

const main = async () => {
    try {
        await client.connect();

        const cursor = await accountsCollection.find(query, options);

        if ((await cursor.countDocuments) === 0) {
            console.log("No documents found!");
        }

        for await (const doc of cursor) {
            console.dir(doc);
        }
    } catch (error) {
        console.error(`Error connecting to the ${dbname} database: ${error}`);
    } finally {
        await client.close();
    }
};

main();
```

Returns.

> { account_id: 'MDB999569384', account_holder: 'James Robson' }
> { account_id: 'MDB999566369', account_holder: 'Giovanny Robson' }
> { account_id: 'MDB999569459', account_holder: 'Ethan Robson' }
> { account_id: 'MDB999569369', account_holder: 'Chris Robson' }
> { account_id: 'MDB999569389', account_holder: 'Charley Robson' }
> { account_id: 'MDB999567869', account_holder: 'Bert Robson' }
> { account_id: 'MDB999566859', account_holder: 'Bert Robson' }
> { account_id: 'MDB999567859', account_holder: 'Andrew Robson' }
> { account_id: 'MDB999569387', account_holder: 'Alan Robson' }

Another complex query.

#### find03.js

```bash
const { MongoClient } = require('mongodb');
const { uri } = require('./atlas_uri');

const client = new MongoClient(uri);
const dbname = 'bank';
const collectionName = 'accounts';

const accountsCollection = client.db(dbname).collection(collectionName);

const query = { balance: { $gt: 3000 } };

const options = {

    // Sort matched documents in ascending order by rating
    sort: { "account_holder": 1 },

    projection: { _id: 0, account_id: 1, account_holder: 1 },
};

const main = async () => {
    try {
        await client.connect();

        const cursor = await accountsCollection.find(query, options);

        if ((await cursor.countDocuments) === 0) {
            console.log("No documents found!");
        }

        for await (const doc of cursor) {
            console.dir(doc);
        }
    } catch (error) {
        console.error(`Error connecting to the ${dbname} database: ${error}`);
    } finally {
        await client.close();
    }
};

main();
```

Returns.

> { account_id: 'MDB999569389', account_holder: 'Charley Robson' }
> { account_id: 'MDB999569369', account_holder: 'Chris Robson' }
> { account_id: 'MDB999566369', account_holder: 'Giovanny Robson' }

### updateMany()

I made a type in one of the account holder's names.

```bash
const { MongoClient } = require('mongodb');
const { uri } = require('./atlas_uri');

const client = new MongoClient(uri);
const dbname = 'bank';
const collectionName = 'accounts';

const accountsCollection = client.db(dbname).collection(collectionName);

// Create a filter to update all movies with a 'G' rating
const filter = { account_holder: "Giovanny Robson" };

const updateDoc = {
    $set: {
        account_holder: 'Giovanni Robson',
    },
};

const main = async () => {
    try {
        await client.connect();

        const result = await accountsCollection.updateMany(filter, updateDoc);
        console.log(`Updated ${result.modifiedCount} documents`);
    } catch (error) {
        console.error(`Error connecting to the ${dbname} database: ${error}`);
    } finally {
        await client.close();
    }
};

main();
```

Returns.

> Updated 1 documents

### deleteOne()

I have added 2 documents with the same ``account_holder`` name. I can delete on the ``account_id`` field which is unique.

#### deleteOne.js

```bash
const { MongoClient } = require('mongodb');
const { uri } = require('./atlas_uri');

const client = new MongoClient(uri);
const dbname = 'bank';
const collectionName = 'accounts';

const accountsCollection = client.db(dbname).collection(collectionName);


const main = async () => {
    try {
        await client.connect();

        // delete the first record with this account_id
        const query = { account_id: 'MDB999566859' };
        const result = await accountsCollection.deleteOne(query);

        if (result.deletedCount === 1) {
            console.log("Successfully deleted one document.");
        } else {
            console.log("No documents matched the query. Deleted 0 documents.");
        }
    } catch (error) {
        console.error(`Error connecting to the ${dbname} database: ${error}`);
    } finally {
        await client.close();
    }
};

main();
```

returns.

> Successfully deleted one document.

### Deleting by _id

I had issues deleting a document by its ``_id``. To get around this you have to build a MongoDB object for the ``_id``.

Change the import statement for ``mongodb``.

```bash
    const { MongoClient, ObjectId } = require('mongodb');
```

Here we add the ``ObjectId`` method.

In out code we have to create a **new** ``ObjectId``.

```bash
    const query = { _id: new ObjectId('65f278b50d791680df86d70c') };
```

This will allow you to delete a document by its ``_id``.

full code.

```bash
const { MongoClient, ObjectId } = require('mongodb');
const { uri } = require('./atlas_uri');

const client = new MongoClient(uri);
const dbname = 'bank';
const collectionName = 'accounts';

const accountsCollection = client.db(dbname).collection(collectionName);

const query = { _id: new ObjectId('65f278b50d791680df86d70c') };

const main = async () => {
    try {
        await client.connect();

        // delete the first record with this account_id
        let result = await accountsCollection.deleteOne(query);

        if (result.deletedCount === 1) {
            console.log("Successfully deleted one document.");
        } else {
            console.log("No documents matched the query. Deleted 0 documents.");
        }
    } catch (error) {
        console.error(`Error connecting to the ${dbname} database: ${error}`);
    } finally {
        await client.close();
    }
};

main();
```

### deleteMany()

We can delete a number of documents from a collection with ``deleteMany()``.

```bash
const query = { balance: { $lt: 2000 } };

const main = async () => {
    try {
        await client.connect();

        let result = await accountsCollection.deleteMany(query);

        if (result.deletedCount > 0) {
            console.log(`Successfully deleted ${result.deletedCount} documents.`);
        } else {
            console.log(`No documents matched the query. Deleted ${result.deletedCount} documents.`);
        }
        ...
```

returns.

> Successfully deleted 2 documents.

**Note:** deleting documents with an empty query will **delete** all documents from a collection.

## Aggregation framework

Is use to build multi-staged queries. The query comprises of three stages.

- $match
- $sort
- $project

Each stage offers output document as input for the next stage. These stages are referred to as an **aggregation pipeline**.

If you can get away with just the ``.find()`` method then use it. Otherwise use the aggregation pipeline framework which is essentially a superset of ``.find()``.

The aggregation framework is more powerful than ``.find()`` and has more primitive operations.

Aggregation frameworks have built in stages for processing documents.

- Finding
- Sorting
- Grouping
- Projecting

These aggregation pipelines make it easier to debug and maintain individual stages. It makes it easier for MongoDB to rewrite and optimise the queries.

An aggregation pipeline can have one or more stages. A stage can use expression variables such as session variables, literal operators and expression objects.

We can think of expression operators as being similar to functions that take in field values as their arguments to compute new field values.

An example.

```bash
[
    {
        _id: ObjectId('65f27156a4c3bfb4d9583b79'),
        account_id: 'MDB999569384',
        account_holder: 'James Robson',
        account_type: 'checking',
        balance: Decimal28('2956'),
        transfers_complete: [
          'TR16523347',
          'TR16523346',
          'TR16523345',
          'TR16523344',
          'TR16523343'
        ]
    }
]
```

Aggregation pipeline.

```bash
    var pipeline = [{
        $match: {
            account_type: 'savings',
            balance: {
                $gt: 1000
            }
        }
    }, {
        $project: {
            _id: 0,
            account_holder: {
                $toUpper: '$account_holder'
            },
            balance: 1,
            Account_type: {
                $toUpper: '$account_type'
            }
        }
    }]
```

We can use this in the following code.

#### aggregation.js

```bash
const { MongoClient } = require('mongodb');
const { uri } = require('./atlas_uri');

const client = new MongoClient(uri);
const dbname = 'bank';

var pipeline = [{
    $match: {
        account_type: 'savings',
        balance: {
            $gt: 1000
        }
    }
}, {
    $project: {
        _id: 0,
        account_holder: {
            $toUpper: '$account_holder'
        },
        balance: 1,
        Account_type: {
            $toUpper: '$account_type'
        }
    }
}]

const main = async () => {
    try {
        await client.connect();

        const database = client.db(dbname);
        const accounts = database.collection('accounts');
        const result = await accounts.aggregate(pipeline);
        for await (const doc of result) {
            console.log(doc);
        }
    } catch (error) {
        console.error(`Error connecting to the ${dbname} database: ${error}`);
    } finally {
        await client.close();
    }
};

main();
```

Returns.

```bash
    {
      balance: 2956.96,
      account_holder: 'ALAN ST JOHN',
      Account_type: 'SAVINGS'
    }
```

Another aggregation example using ``$match`` and ``$group``.

#### aggregation02.js

```bash
const { MongoClient } = require('mongodb');
const { uri } = require('./atlas_uri');

const client = new MongoClient(uri);
const dbname = 'bank';

var pipeline = [{
    $match: { balance: { $gt: 1000 } }
}, {
    $group: {
        _id: '$account_type',
        total_balance: { $sum: '$balance' },
        average_balance: { $avg: '$balance' },
    },
}]

const main = async () => {
    try {
        await client.connect();

        const database = client.db(dbname);
        const accounts = database.collection('accounts');
        const result = await accounts.aggregate(pipeline);
        for await (const doc of result) {
            console.log(doc);
        }
    } catch (error) {
        console.error(`Error connecting to the ${dbname} database: ${error}`);
    } finally {
        await client.close();
    }
};

main();
```

Results:

```bash
    { _id: 'savings', total_balance: 2956.96, average_balance: 2956.96 }
    {
      _id: 'checking',
      total_balance: 31668.9,
      average_balance: 3166.8900000000003
    }
```

Now we will do another aggregation example with the ``$match``, ``$sort`` and ``$project``.

``$sort`` uses 1 for ascending sorts and -1 for descending sorts.

The projection creates a new computed variable ``gbp_balance`` where we create a balance that is converted to British pounds.

#### aggregation03.js

```bash
const { MongoClient } = require('mongodb');
const { uri } = require('./atlas_uri');

const client = new MongoClient(uri);
const dbname = 'bank';

var pipeline = [{
    $match: { account_type: 'checking', balance: { $gt: 1000 } }
}, {
    $sort: { balance: -1 }
}, {
    $project: {
        _id: 0,
        account_id: 1,
        account_type: 1,
        gbp_balance: { $divide: ['$balance', 1.5] },
    },
},
]

const main = async () => {
    try {
        await client.connect();

        const database = client.db(dbname);
        const accounts = database.collection('accounts');
        const result = await accounts.aggregate(pipeline);
        for await (const doc of result) {
            console.log(doc);
        }
    } catch (error) {
        console.error(`Error connecting to the ${dbname} database: ${error}`);
    } finally {
        await client.close();
    }
};

main();
```

Results.

```bash
    {
      account_id: 'MDB999569369',
      account_type: 'checking',
      gbp_balance: 4371.26
    }
    {
      account_id: 'MDB999569389',
      account_type: 'checking',
      gbp_balance: 3304.5933333333337
    }
    {
      account_id: 'MDB999569384',
      account_type: 'checking',
      gbp_balance: 1971.26
    }
    {
      account_id: 'MDB999569459',
      account_type: 'checking',
      gbp_balance: 1504.5933333333332
    }
```

## Mongoose

All of these examples have been created in **MongoDB** code and there is another way to reduce the amount of typing. We can use the Mongoose package which has a lot of options that can be used with Node.js.

See **MongooseClient** for the Mongoose versions of this code.

### A Mongoose example

For this example we need to install some more packages.

Install the Mongoose package.

```bash
    npm install mongoose
```

Install the Dotenv package.

```bash
    npm install dotenv
```

#### ./models/bankModel.js

```bash
    const mongoose = require('mongoose');

    const bankSchema = new mongoose.Schema({
        account_id: String,
        account_type: String,
        balance: Number
    });

    module.exports = mongoose.model('Account', bankSchema);
```

This is the schema that Mongoose requires for modelling the data.

#### mongoose.js

```bash
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const Account = require('./models/bankModel');

dotenv.config({ path: './config/dev.env' });

const uri = process.env.MONGODB_URL;
mongoose.connect(uri);
const db = mongoose.connection;
db.on('error', console.error.bind(console, 'MongoDB connection error:'));
db.once('open', () => console.log('Connected to MongoDB'));

const pipeline = [
    {
        $match: { account_type: 'savings', balance: { $gt: 1000 } }
    },
    {
        $sort: { balance: -1 }
    },
    {
        $project: {
            _id: 0,
            account_id: 1,
            account_type: 1,
            gbp_balance: { $divide: ['$balance', 1.5] },
        },
    },
];

const main = async () => {
    try {
        const result = await Account.aggregate(pipeline);

        if (result.length === 0) {
            console.log("No documents found!");
        } else {
            result.forEach((doc) => {
                console.log(doc);
            });
        }
    } catch (error) {
        console.error(`Error querying the database: ${error}`);
    } finally {
        mongoose.connection.close();
    }
};

main();
```

Returns.

```bash
    Connected to MongoDB...
    {
      account_id: 'MDB999569532',
      account_type: 'savings',
      gbp_balance: 1971.3066666666666
    }
    {
      account_id: 'MDB999569567',
      account_type: 'savings',
      gbp_balance: 1971.26
    }
    {
      account_id: 'MDB999569678',
      account_type: 'savings',
      gbp_balance: 1065.3333333333333
    }
```

### How does Mongoose work?

In the previous example, Mongoose knows how to find documents in the ``accounts`` collection based on the model definition and the connection to the MongoDB database.

**Model Definition:** When you define a Mongoose model using mongoose.model, you provide a name for the model (in this case, '``Account``') and a schema (in this case, ``accountSchema``). This associates the model with a specific MongoDB collection. In this case, the ``Account`` model is associated with the ``accounts`` collection.

**Connection to the MongoDB Database:** When you establish a connection to the MongoDB database using mongoose.connect, Mongoose internally maintains a connection pool and keeps track of the schemas and models associated with the database. This allows Mongoose to know which collection corresponds to each model.

So, when you perform operations using the ``Account`` model (e.g., querying documents), Mongoose automatically translates those operations into MongoDB queries targeting the ``accounts`` collection.
