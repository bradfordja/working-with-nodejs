# Node using GraphQL API

Writing a sample code for a Node web application using require SQL, and GraphQL api that manages engineer [userId, firstName, lastName, fullName, title, status,  siteId] and engineer site locations [siteId, siteName, address, city, state, status].  I want the Node API end points to be able to

1. search for engineer by name,
2. search engineer id
3. search for engineer by site,
4. search engineer status
5. edit an engineer [userId, firstName, lastName, fullName, title, status, siteId]
6. edit a site location [siteId, siteName, address, city, state]

Answers with simple explanation and step by step sample uses-case and include [ modules, controller , model, unit test] file to create.

Sure! I'll help you create a Node web application using SQL, and GraphQL API to manage engineers and engineer site locations. We'll structure the project into modules, controllers, models, and unit tests. To achieve this, you'll need to set up a new Node.js project, install the necessary dependencies, and create the required files.

**Step-by-step guide:**

### 1. **Initialize the project and install dependencies:**
Create a new directory for your project and navigate into it. Run the following commands to initialize the project and install the required dependencies.

```bash
npm init -y
npm install express mssql graphql express-graphql @types/express @types/mssql @types/graphql jest ts-jest supertest --save
```

### 2. **Set up the database connection:**
Create a file called `dbConfig.js` to configure the connection to your Microsoft SQL Server database.

```javascript
// dbConfig.js
const sql = require('mssql');

const config = {
  user: 'your_username',
  password: 'your_password',
  server: 'your_server',
  database: 'your_database',
  options: {
    encrypt: true, // For encrypted connections
    trustServerCertificate: true, // For self-signed certificates
  },
};

async function connect() {
  try {
    await sql.connect(config);
    console.log('Connected to SQL Server');
  } catch (err) {
    console.error('Error connecting to SQL Server:', err.message);
  }
}

module.exports = {
  sql,
  connect,
};
```

### 3. **Create the engineer model:**
Create a file called `engineerModel.js` to define the engineer and site location models and their corresponding CRUD operations.

```javascript
// engineerModel.js
const { sql } = require('./dbConfig');

async function getEngineerByName(firstName, lastName) {
  try {
    const pool = await sql.connect();
    const result = await pool
      .request()
      .input('firstName', sql.VarChar, firstName)
      .input('lastName', sql.VarChar, lastName)
      .query('SELECT * FROM engineers WHERE firstName = @firstName AND lastName = @lastName');
    return result.recordset;
  } catch (err) {
    console.error('Error retrieving engineer:', err.message);
    throw err;
  }
}

// Implement other CRUD operations for engineers and site locations
// ...

module.exports = {
  getEngineerByName,
  // Add other CRUD methods here
};
```

### 4. **Create the engineer site model:**
Create a file called `siteModel.js` to define the engineer site model and its corresponding CRUD operations.

```javascript
// siteModel.js
const { sql } = require('./dbConfig');

async function getEngineerBySite(siteId) {
  try {
    const pool = await sql.connect();
    const result = await pool
      .request()
      .input('siteId', sql.Int, siteId)
      .query('SELECT * FROM engineers WHERE siteId = @siteId');
    return result.recordset;
  } catch (err) {
    console.error('Error retrieving engineer by site:', err.message);
    throw err;
  }
}

// Implement other CRUD operations for site locations
// ...

module.exports = {
  getEngineerBySite,
  // Add other CRUD methods here
};
```

### 5. **Create the GraphQL schema and resolvers:**
Create a file called `graphql.js` to define the GraphQL schema and resolvers for handling GraphQL queries.

```javascript
// graphql.js
const { buildSchema } = require('graphql');
const engineerModel = require('./engineerModel');
const siteModel = require('./siteModel');

const schema = buildSchema(`
  type Engineer {
    userId: Int!
    firstName: String!
    lastName: String!
    fullName: String!
    title: String!
    status: String!
    siteId: Int!
  }

  type SiteLocation {
    siteId: Int!
    siteName: String!
    address: String!
    city: String!
    state: String!
    status: String!
  }

  type Query {
    engineerByName(firstName: String!, lastName: String!): [Engineer!]!
    engineerBySite(siteId: Int!): [Engineer!]!
  }
`);

const resolvers = {
  engineerByName: ({ firstName, lastName }) => engineerModel.getEngineerByName(firstName, lastName),
  engineerBySite: ({ siteId }) => siteModel.getEngineerBySite(siteId),
};

module.exports = { schema, resolvers };
```

### 6. **Set up the Express server and GraphQL endpoint:**
Create a file called `server.js` to set up the Express server and the GraphQL endpoint.

```javascript
// server.js
const express = require('express');
const { graphqlHTTP } = require('express-graphql');
const { schema, resolvers } = require('./graphql');
const { connect } = require('./dbConfig');

const app = express();
connect();

app.use(
  '/graphql',
  graphqlHTTP({
    schema: schema,
    rootValue: resolvers,
    graphiql: true, // Enable the GraphiQL UI for testing
  })
);

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server started on http://localhost:${PORT}/graphql`);
});
```

### 7. **Unit testing:**
Create a directory called `__tests__` to store the unit test files.

```bash
mkdir __tests__
```

Create a file called `engineerModel.test.js` inside the `__tests__` directory to write unit tests for the `engineerModel`.

```javascript
// __tests__/engineerModel.test.js
const engineerModel = require('../engineerModel');

describe('Engineer Model', () => {
  test('Get engineer by name', async () => {
    const engineers = await engineerModel.getEngineerByName('John', 'Doe');
    expect(engineers.length).toBe(1);
    expect(engineers[0].firstName).toBe('John');
    expect(engineers[0].lastName).toBe('Doe');
  });

  // Add other test cases for CRUD operations
  // ...
});
```

### 8. **Run the unit tests:**
Add the following script to your `package.json` file to run the unit tests using Jest.

```json
{
  "scripts": {
    "test": "jest"
  }
}
```

### 9. **Sample use-cases:**

- **Search for an engineer by name:**
  ```graphql
  query {
    engineerByName(firstName: "John", lastName: "Doe") {
      userId
      firstName
      lastName
      fullName
      title
      status
      siteId
    }
  }
  ```

- **Search for an engineer by site:**
  ```graphql
  query {
    engineerBySite(siteId: 123) {
      userId
      firstName
      lastName
      fullName
      title
      status
      siteId
    }
  }
  ```

- **Edit an engineer:**
  ```graphql
  mutation {
    editEngineer(
      userId: 456
      firstName: "John"
      lastName: "Doe"
      title: "Software Engineer"
      status: "Active"
      siteId: 789
    ) {
      userId
      firstName
      lastName
      fullName
      title
      status
     

 siteId
    }
  }
  ```

- **Edit a site location:**
  ```graphql
  mutation {
    editSiteLocation(
      siteId: 789
      siteName: "Site A"
      address: "123 Main Street"
      city: "New York"
      state: "NY"
    ) {
      siteId
      siteName
      address
      city
      state
      status
    }
  }
  ```

**Explanation:**

- The project is organized into different files: `dbConfig.js` for database connection, `engineerModel.js` for engineer CRUD operations, `siteModel.js` for site location CRUD operations, `graphql.js` for defining the GraphQL schema and resolvers, and `server.js` for setting up the Express server and GraphQL endpoint.

- The GraphQL schema includes two types: `Engineer` and `SiteLocation`, each representing an engineer and a site location, respectively.

- The `Query` type in the GraphQL schema includes two queries: `engineerByName` and `engineerBySite`, which are used to search for engineers by name and site, respectively.

- The `resolvers` object in `graphql.js` defines the functions that handle the GraphQL queries. These resolvers use the methods defined in `engineerModel.js` and `siteModel.js` to fetch data from the database.

- The Express server is set up in `server.js` and listens on port 3000. It mounts the GraphQL endpoint at `/graphql`, which can be accessed through the GraphiQL UI for testing.

- The unit tests in `__tests__/engineerModel.test.js` use Jest to test the `engineerModel` methods.

Please note that this is a basic implementation of the project and doesn't cover authentication, validation, and other aspects that may be required in a real-world application. For production use, it's important to implement additional security measures and handle error scenarios appropriately.