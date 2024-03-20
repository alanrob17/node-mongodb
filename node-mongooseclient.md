# Node Mongoose client examples

**Source:** D:\Project-Examples\node-mongodb\mongooseClient

## Setup

Initialise the project.

```bash
    npm init -y
```

Install the Mongoose package.

```bash
    npm install mongoose
```

Install the Dotenv package.

```bash
    npm install dotenv
```

### Testing

#### app.js

```bash
    const mongoose = require('mongoose');
    const dotenv = require('dotenv');

    dotenv.config({ path: './config/dev.env' });

    console.log(process.env.MONGODB_URL);
```

Returns.

> mongodb://alan:test123@localhost:27018/record-db?authSource=admin

This tells us our **env** file is working.

## Creating a model

Mongoose works on a model of your data. In our case we have a ``bankModel`` based on our accounts collection.

### bankModel.js

```bash
    const mongoose = require('mongoose');

    const bankSchema = new mongoose.Schema({
        account_id: String,
        account_holder: String,
        account_type: String,
        balance: Number
    });

    module.exports = mongoose.model('Account', bankSchema);
```

You make a schema of your collection. The ``exports`` statement exports a model (``Account``) of your collection.

**Note:** Mongoose associates your ``accounts`` collection with your ``Account`` model. You don't have to refer to your ``accounts`` collection.

In your JavaScript code you access your mode l with.

```bash
    const Account = require('./models/bankModel');
```

## Inserting documents

### insertOne.js

```bash
    const mongoose = require('mongoose');
    const dotenv = require('dotenv');
    const Account = require('./models/bankModel');

    dotenv.config({ path: './config/dev.env' });

    const uri = process.env.MONGODB_URL;

    const main = async () => {
        try {
            await mongoose.connect(uri);

            console.log('Connected to MongoDB...\n');

            // Create a new account document
            const account = new Account({
                account_id: 'MDB999569889',
                account_holder: 'Ethan Robson',
                account_type: 'savings',
                balance: 245.75
            });

            // Save the account document to the database
            const result = await account.save();
            console.log(result._id);
        } catch (error) {
            console.error(`Error: ${error}`);
        } finally {
            await mongoose.connection.close();
        }
    };

    main();
```

You can insert a number of documents in one process.

### insertMany.js

```bash
    const mongoose = require('mongoose');
    const dotenv = require('dotenv');
    const Account = require('./models/bankModel');

    dotenv.config({ path: './config/dev.env' });

    const uri = process.env.MONGODB_URL;

    const main = async () => {
        try {
            await mongoose.connect(uri);

            console.log('Connected to MongoDB...\n');

            const sampleAccounts = [
                {
                    account_id: 'MDB999569985',
                    account_holder: 'Bert Smith',
                    account_type: 'savings',
                    balance: 1745.67
                },
                {
                    account_id: 'MDB999569986',
                    account_holder: 'Andrew Jones',
                    account_type: 'savings',
                    balance: 1985.99
                },
                {
                    account_id: 'MDB999569987',
                    account_holder: 'Bob Mongo',
                    account_type: 'savings',
                    balance: 456.75
                }
            ];

            const accounts = await Account.insertMany(sampleAccounts);
            accounts.forEach(account => {
                console.log(account._id);
            });
        } catch (error) {
            console.error(`Error: ${error}`);
        } finally {
            await mongoose.connection.close();
        }
    };

    main();
```

## Finding documents

### app.js

```bash
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const Account = require('./models/bankModel');

dotenv.config({ path: './config/dev.env' });

const uri = process.env.MONGODB_URL;

const main = async () => {
    try {
        await mongoose.connect(uri);

        console.log('Connected to MongoDB...\n');

        const result = await Account.find({ balance: { $gt: 1000 } });

        result.forEach(item => {
            console.log(item);
        });
    } catch (error) {
        console.error(`Error: ${error}`);
    } finally {
        await mongoose.connection.close();
    }
};

main();
```

Returns.

```bash
    Connected to MongoDB...

    {
      _id: new ObjectId('65f91fed4a2f156e0a7693d3'),
      account_id: 'MDB999569234',
      account_holder: 'James Robson',
      account_type: 'savings',
      balance: 2345.99,
      __v: 0
    }
    {
      _id: new ObjectId('65f920134c813decc7e3031b'),
      account_id: 'MDB999569789',
      account_holder: 'Charley Robson',
      account_type: 'savings',
      balance: 4562.55,
      __v: 0
    }
    ...
```

### find01.js

```bash
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const Account = require('./models/bankModel');

dotenv.config({ path: './config/dev.env' });

const uri = process.env.MONGODB_URL;

const query = { account_holder: { $regex: /Robson/ }, balance: { $gt: 1000 } };

const main = async () => {
    try {
        await mongoose.connect(uri);

        console.log('Connected to MongoDB...\n');

        const result = await Account.find(query);

        result.forEach(item => {
            console.log(item);
        });
    } catch (error) {
        console.error(`Error: ${error}`);
    } finally {
        await mongoose.connection.close();
    }
};

main();
```

returns.

```bash
    {
      _id: new ObjectId('65f91fed4a2f156e0a7693d3'),
      account_id: 'MDB999569234',
      account_holder: 'James Robson',
      account_type: 'savings',
      balance: 2345.99,
      __v: 0
    }
    {
      _id: new ObjectId('65f920134c813decc7e3031b'),
      account_id: 'MDB999569789',
      account_holder: 'Charley Robson',
      account_type: 'savings',
      balance: 4562.55,
      __v: 0
    }
    ...
```

This is a slightly more complex query but using ``find()`` in Mongoose is limited to relatively simple searches. For more complex searching use an aggregation framework.

## Aggregation framework

See file:///C:/Git/node-mongodb/node-mongodbclient-f.html for notes on using aggregations.

### aggregation01.js

```bash
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const Account = require('./models/bankModel');

dotenv.config({ path: './config/dev.env' });

const uri = process.env.MONGODB_URL;

const main = async () => {
    try {
        const connection = mongoose.connection;

        connection.on('error', console.error.bind(console, 'MongoDB connection error:'));
        connection.once('open', async () => {
            console.log('Connected to MongoDB...\n');

            try {
                const accounts = await Account.aggregate([
                    { $match: { account_holder: { $regex: /Robson/ }, balance: { $gt: 1000 } } },
                    { $sort: { account_holder: 1 } },
                    { $project: { _id: 0, account_holder: 1, account_type: 1, balance: 1 } }
                ]);

                accounts.forEach(account => {
                    console.log(account);
                });
            } catch (error) {
                console.error(`Error querying the database: ${error}`);
            } finally {
                await mongoose.connection.close();
            }
        });

        await mongoose.connect(uri);
    } catch (error) {
        console.error(`Error: ${error}`);
    }
};

main();
```

Returns.

```bash
    Connected to MongoDB...

    {
      account_holder: 'Andrew Robson',
      account_type: 'savings',
      balance: 2345.75
    }
    {
      account_holder: 'Charley Robson',
      account_type: 'savings',
      balance: 4562.55
    }
    ...
```

This allows us to use ``$match``, ``$sort`` and ``$project`` to create more complex queries.

### aggregation02.js

```bash
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const Account = require('./models/bankModel');

dotenv.config({ path: './config/dev.env' });

const uri = process.env.MONGODB_URL;

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
        const connection = mongoose.connection;

        connection.on('error', console.error.bind(console, 'MongoDB connection error:'));
        connection.once('open', async () => {
            console.log('Connected to MongoDB...\n');

            try {
                const accounts = await Account.aggregate(pipeline);

                accounts.forEach(account => {
                    console.log(account);
                });
            } catch (error) {
                console.error(`Error querying the database: ${error}`);
            } finally {
                await mongoose.connection.close();
            }
        });

        await mongoose.connect(uri);
    } catch (error) {
        console.error(`Error: ${error}`);
    }
};

main();
```

Returns.

```bash
    {
      balance: 2345.99,
      account_holder: 'JAMES ROBSON',
      Account_type: 'SAVINGS'
    }
    {
      balance: 4562.55,
      account_holder: 'CHARLEY ROBSON',
      Account_type: 'SAVINGS'
    }
    ...
```

In this case we can modify the data being returned.

### aggregation03.js

```bash
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const Account = require('./models/bankModel');

dotenv.config({ path: './config/dev.env' });

const uri = process.env.MONGODB_URL;

var pipeline = [{
    $match: { account_type: 'savings', balance: { $gt: 1000 } }
}, {
    $sort: { account_holder: 1 }
}, {
    $project: {
        _id: 0,
        account_id: 1,
        account_holder: 1,
        account_type: 1,
        gbp_balance: { $divide: ['$balance', 1.5] },
    },
},
]

const main = async () => {
    try {
        const connection = mongoose.connection;

        connection.on('error', console.error.bind(console, 'MongoDB connection error:'));
        connection.once('open', async () => {
            console.log('Connected to MongoDB...\n');

            try {
                const accounts = await Account.aggregate(pipeline);

                accounts.forEach(account => {
                    console.log(account);
                });
            } catch (error) {
                console.error(`Error querying the database: ${error}`);
            } finally {
                await mongoose.connection.close();
            }
        });

        await mongoose.connect(uri);
    } catch (error) {
        console.error(`Error: ${error}`);
    }
};

main();
```

Returns.

```bash
    {
      account_id: 'MDB999569522',
      account_holder: 'Andrew Robson',
      account_type: 'savings',
      gbp_balance: 1563.8333333333333
    }
    {
      account_id: 'MDB999569789',
      account_holder: 'Charley Robson',
      account_type: 'savings',
      gbp_balance: 3041.7000000000003
    }
    ...
```

In this case we are computing a value by converting the balance to British pounds.

## Updating documents

### updateOne-01.js

We can use ``filter`` and ``update`` values in the update command.

```bash
    const mongoose = require('mongoose');
    const dotenv = require('dotenv');
    const Account = require('./models/bankModel');

    dotenv.config({ path: './config/dev.env' });

    const uri = process.env.MONGODB_URL;

    const filter = { account_id: 'MDB999569678' };
    const update = { $set: { account_holder: 'Alan Robson' } };

    const main = async () => {
        try {
            await mongoose.connect(uri);

            console.log('Connected to MongoDB...\n');

            const account = await Account.updateOne(filter, update);
            console.log(account);
        } catch (error) {
            console.error(`Error: ${error}`);
        } finally {
            await mongoose.connection.close();
        }
    };

    main();
```

### updateOne-02.js

```bash
    const mongoose = require('mongoose');
    const { ObjectId } = require('mongodb'); // Import ObjectId from 'mongodb'
    const dotenv = require('dotenv');
    const Account = require('./models/bankModel');

    dotenv.config({ path: './config/dev.env' });

    const uri = process.env.MONGODB_URL;

    const filter = { _id: new ObjectId('65f91f7746cfa71788e09ec2') };
    const update = { $set: { balance: 799.00 } };

    const main = async () => {
        try {
            await mongoose.connect(uri);

            console.log('Connected to MongoDB...\n');

            const account = await Account.updateOne(filter, update);
            console.log(account);
        } catch (error) {
            console.error(`Error: ${error}`);
        } finally {
            await mongoose.connection.close();
        }
    };

    main();
```

When we want to work with an ObjectId value in a query we have to use the **mongodb** package. We also have to use ``new`` to create a new object in the filter.

### updateMany.js

```bash
    const mongoose = require('mongoose');
    const dotenv = require('dotenv');
    const Account = require('./models/bankModel');

    dotenv.config({ path: './config/dev.env' });

    const uri = process.env.MONGODB_URL;

    const filter = { account_holder: { $regex: /Mongo/ } };
    const update = { $set: { account_holder: 'Bob Johnson' } };

    const main = async () => {
        try {
            await mongoose.connect(uri);

            console.log('Connected to MongoDB...\n');

            const account = await Account.updateMany(filter, update);
            console.log(account);
        } catch (error) {
            console.error(`Error: ${error}`);
        } finally {
            await mongoose.connection.close();
        }
    };

    main();
```

## Deleting documents

### deleteOne.js

```bash
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const Account = require('./models/bankModel');

dotenv.config({ path: './config/dev.env' });

const uri = process.env.MONGODB_URL;

const filter = { account_holder: 'Luigi Robson' };

const main = async () => {
    try {
        await mongoose.connect(uri);

        console.log('Connected to MongoDB...\n');

        const deletionResult = await Account.deleteOne(filter);

        console.log('Deletion result:', deletionResult);

        console.log('Number of documents deleted:', deletionResult.deletedCount);
    } catch (error) {
        console.error(`Error: ${error}`);
    } finally {
        await mongoose.connection.close();
    }
};

main();
```

Returns.

```bash
    Connected to MongoDB...

    Deletion result: { acknowledged: true, deletedCount: 1 }
    Number of documents deleted: 1
```

Delete a document using an ``ObjectId``.

### deleteOne-02.js

```bash
const mongoose = require('mongoose');
const { ObjectId } = require('mongodb');
const dotenv = require('dotenv');
const Account = require('./models/bankModel');

dotenv.config({ path: './config/dev.env' });

const uri = process.env.MONGODB_URL;

const filter = { _id: new ObjectId('65f933f9559dabc0f54c685f') };

const main = async () => {
    try {
        await mongoose.connect(uri);

        console.log('Connected to MongoDB...\n');

        const deletionResult = await Account.deleteOne(filter);

        console.log('Deletion result:', deletionResult);

        console.log('Number of documents deleted:', deletionResult.deletedCount);
    } catch (error) {
        console.error(`Error: ${error}`);
    } finally {
        await mongoose.connection.close();
    }
};

main();
```

### deleteMany.js

```bash
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const Account = require('./models/bankModel');

dotenv.config({ path: './config/dev.env' });

const uri = process.env.MONGODB_URL;

const filter = { account_holder: 'Bert Smith' };

const main = async () => {
    try {
        await mongoose.connect(uri);

        console.log('Connected to MongoDB...\n');

        const deletionResult = await Account.deleteMany(filter);

        console.log('Deletion result:', deletionResult);

        console.log('Number of documents deleted:', deletionResult.deletedCount);
    } catch (error) {
        console.error(`Error: ${error}`);
    } finally {
        await mongoose.connection.close();
    }
};

main();
```
