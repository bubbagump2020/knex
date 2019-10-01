# Knex

## Outline

- Managing a database schema with Knex
- Seeding a database with Knex
- Querying for data with Knex

## Lesson

### Managing a database schema with Knex

Let's start by installing Knex, and a driver for our specific database, sqlite3:

<br/>

> Terminal  

```
npm install knex
npm install sqlite3
```

<br/>

Next, let's create a configuration file for database connection:

<br />

> Terminal  

```
npx knex init
```

<br />

This should create a file named `knexfile.js` in your project, which will contain settings for creating a connection to your database; the default settings should be fine for now.

Now that we can connect to the database, let's make some migrations to create a table schema.

Knex has it's own command line interface (CLI) that you can use to generate migrations. Node.js CLIs are generally configured to be run with NPX (Node Package eXecutor); we can use `npx` and knex to create a migration named "create_reviews" like so:

<br />

> Terminal

```javascript
npx knex migration:make create_users_table
```

This should have created a migrations folder in your project, with one file inside. Open that file. It should looks something like this:

<br />

> 20190930182507_create_users_table.js (your number will be different)

```javascript
exports.up = function(knex) {
   
};

exports.down = function(knex) {
  
};
```

<br />

This default migration is exporting two functions- the `up` function would be used to update the table schema.

The `down` function would be used to reverse the update that `up` makes. This gives us a way to run and rollback our migration later at will. 

Both functions receive `knex` as an argument, which is an object that contains all of the methods and properties knex creates for us to interact with the database. `knex.schema` is _another_ object with methods specifically for managing a database schema.

Lets use the `createTable` method to create our users table:

<br />

> 20190930182507_create_users_table.js (your number will be different)

```javascript
exports.up = function(knex) {
   return knex.schema.createTable('users', (table) => {
        table.increments('id');
        table.string('firstName');
       	table.string('lastName');
        table.integer('age');
    })
};

exports.down = function(knex) {
  return knex.schema.dropTable('users')
};
```

<br />

Here, we pass `createTable` the name of the table we want to create, and a callback, which we can use to specify the columns the table should have.

`table.increments('id')` creates an auto incrementing ID for the table.

`table.string('firstName')` creates a column with a string datatype and the name 'firstName', etc.

In our `down` method, we'll use the `dropTable` method incase this migration ever needs to be reversed.

It's important to note that in both cases, we **return** the migration operations we write. If we forget to return the migration methods, they won't work later when we try to run the migration.

You can see a full list of the datatypes you can use here: <http://knexjs.org/#Schema-Building>

Now that we've created our migration, let's use `npx` to run it:

<br />

> Terminal

```javascript
npx knex migrate:up
```

<br />

Running this should have created a file in your project called `dev.sqlite3`. If you have a database visualizer like DB Browser, use it to check the schema of the database, among a few tables which `knex` creates automatically, you should find our users table. Close DB Browser before moving forward to avoid write conflicts.



###  Seeding a database with Knex

Now that our table has structure, let's add some data, using a seed file.

The knex CLI can be used to make a seed file:

<br />

> Terminal

```
npx knex seed:make user_seeds
```

<br />

This will create a `seeds` folder with a `user_seeds.js` file. It should look like this:

<br />

> seeds/user_seeds.js

```javascript
exports.seed = function(knex) {
  // Deletes ALL existing entries
  return knex('table_name').del()
    .then(function () {
      // Inserts seed entries
      return knex('table_name').insert([
        {id: 1, colName: 'rowValue1'},
        {id: 2, colName: 'rowValue2'},
        {id: 3, colName: 'rowValue3'}
      ]);
    });
};
```

<br />

The default seed file deletes all rows from an example table, and then inserts some new rows. Because interacting with the database is **asynchronous**, this code uses promises and `.then` to make sure things happen in the proper sequence.

We can refactor it a little bit to use `async/await`:

<br />

> seeds/user_seeds.js

```javascript
exports.seed = async function(knex) {
     
   // Deletes ALL existing entries
   await knex('table_name').del()
   
   // Inserts seed entries
   await knex('table_name').insert([
        {id: 1, colName: 'rowValue1'},
        {id: 2, colName: 'rowValue2'},
        {id: 3, colName: 'rowValue3'}
      ]);
   });

};
```

<br />

Finally, let's change the data to suite our project, using the `users` table we created earlier and the columns we defined:

<br />

> seeds/user_seeds.js

```javascript
exports.seed = async function(knex) {
     
   // Deletes ALL existing users
   await knex('users').del()
   
   // Inserts seed users
   await knex('users').insert([
        { firstName: 'John', lastName: 'Doe', age: 26 },
        { firstName: 'Jennifer', lastName: 'Smith', age: 30 }
      ]);
   });

};
```

<br />

Let's run the seed file from the terminal:

<br />

> Terminal

```
npx knex seed:run
```

<br />

Now, if you re-open the database with DB Browser, we should be able to see John and Jennifer in our users table.

Breaking down the code in our seed file, using `knex` as a funciton, we selected the users table. That returned an object with **alot** of methods (you can see all of them here: <http://knexjs.org/#Builder>). We used the `del` method, which, when invoked without arguments, deletes all rows from the table.

We're doing mostly the same thing down below, still using `knex` to select a specific table, but this time we're using the `insert` method, and passing in an array of objects to be added as rows to our table.

Now that we've created and seeded our table, let's see how we can query rows from our table



### Querying a database with Knex

Let's write a program to pull information out of our database and log it in `index.js`.

First, we need to require `knex`, give it our connection settings so that it can connect to our database.

Previously, in our seeds and migration files, knex did this automatically. Now that we are writing our own script, we will have to provide this configuration manually.

In `index.js`, let's require the database settings from the `knexfile` we created earlier:

<br />

> index.js

```javascript
const databaseSettings = require('./knexfile.js')
```

<br />

If we open `knexfile.js`, we see that it actually has settings for several database connections:

<br />

> knexfile.js

```javascript
module.exports = {

  development: {
    client: 'sqlite3',
    connection: {
      filename: './dev.sqlite3'
    }
  },

  staging: {
    client: 'postgresql',
    connection: {
...
```

<br />Each of these settings are meant to be used in different environments, depending on whether our app is being developed, tested, or in production. Right now, we're developing, so let's selecte the development settings:

<br />

> index.js

```javascript
const databaseSettings = require('./knexfile.js').development
```

<br />

Now we can require `knex` using our settings:

<br />

> index.js

```javascript
const databaseSettings = require('./knexfile.js').development
const knex = require('knex')(databaseSettings.development)
```

<br />

This is the same `knex` object we've used in our seeds and migrations. As you develop with `knex`, it will be critical that you read and learn from the documentation, to query data from your database in a way that matches your specific use case. For now, we'll look at one common use case, selecting all rows from a table: 

<br />

> index.js

```javascript
const databaseSettings = require('./knexfile.js')
const knex = require('knex')(databaseSettings.development)

let users = await knex('users').select()
console.log(users)
```

<br />

As before, `knex` runs asyncronously, so we'll need to `await` the users it's returning.

This code, however, will throw an error, because **`await` can only be used inside of `async` functions.**

We could use `.then` instead, or we could wrap our code in an `async` function and then invoke it:

<br />

> index.js

```javascript
const databaseSettings = require('./knexfile.js')
const knex = require('knex')(databaseSettings.development)
async function logUsers(){
    let users = await knex('users').select()
    console.log(users)
}
logUsers()
```

<br />

When you run this file (`node index.js`), you should see an array of users from the database in your terminal.

We could also select more specific data using a where clause:

<br />

> index.js

```javascript
const databaseSettings = require('./knexfile.js')
const knex = require('knex')(databaseSettings.development)
async function logUsers(){
    let users = await knex('users').where({ name: 'Jennifer' }).select()
    console.log(users)
}
logUsers()
```

<br />



## Check your Understanding

Now that we know how to create, seed, and query tables, let's practice building a full CRUD RESTful API:

1. Create a new project folder
2. Install `express` to manage HTTP routing and `knex` to manage your database
3. Follow the steps in this project to create `fruits` table:
   1. Every fruit will have a name
   2. Every fruit will have a color
4. Create an `index.js` in the root of your project
5. Inside of `index.js`, require `express` and `knex`, and use them together to create the following RESTful endpoints:

- `index` 
  - HTTP Method: 'GET'
  - Path: '/fruits'
  - Should: return an array of all fruit from the database
- `show` 
  - HTTP Method: 'GET'
  - Path: '/fruits/:id'
  - Should: return a single fruit whose `id` matches the id in the path
- `create` 
  - HTTP Method: 'POST'
  - Path: '/fruits'
  - Body:
    - name
    - color
  - Should: create a new fruit in the database with the name and color provided
- `update` 
  - HTTP Method: 'PATCH'
  - Path: '/fruits/:id'
  - Body:
    - name
    - color
  - Should: update one of the fruit in the database with the name and color provided
- `destroy`
  - HTTP Method: 'DELETE'
  - Path: '/fruits/:id'
  - Should: Delete fruit from the database
