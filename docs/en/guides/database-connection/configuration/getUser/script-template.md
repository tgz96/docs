The script will be used in these user cases: 

- When a user tries to register, if your database script finds this user's information, it will reject the registration requirement. 
- When OIDC protocol needs to exchange user information. 
- When someone uses console or API to check user information. 

This script is required in both LAZY_MIGRATION and CUSTOM_USER_STORE mode.


### Function Definition

Here is the definition of the `getUser` function:

```javascript
async function getUser(query, context) {
  // The first argument query contains following contents:
  // query.id
  // query.email
  // query.username
  // query.phone

  // The Second argument context contains information about the authentication context.
  // see http://core.authing.cn/connections/custom-db/config-custom-db-connection.html for more information.

  // This script should retrieve a user profile from your existing database,
  // without authenticating the user.
  // It is used to check if a user exists before executing flows that do not
  // require authentication (signup and password reset).
  //
  // There are three ways this script can finish:
  // 1. A user was successfully found. The profile should be in the following
  // format: https://docs.authing.co/user/profile.html .
  //    return profile
  // 2. A user was not found
  //     return null
  // 3. Something went wrong while trying to reach your database:
  //     throw new Error("my error message")

  const msg =
    'Please implement the Find User script for this database connection ';
  throw new Error(msg);
}
```

| Parameter         | Type   | Nullable | Explanation                                                      |
| :------------- | :----- | :------- | :--------------------------------------------------------- |
| query          | object | false    | Query condition                                                  |
| query.id       | string | ture     | User's ID. This parameter can be empty.                                |
| query.email    | string | ture     | User's email. This parameter can be empty.                                     |
| query.phone    | string | true     | User's telephone number. This parameter can be empty.                                   |
| query.username | string | true     | User's username. This parameter can be empty.                                   |
| context        | object | true     | Requiring context                                          |

The context also includes the following information:

| Property Name           | Type   | Explanation                                                                                                        |
| :--------------- | :----- | :---------------------------------------------------------------------------------------------------------- |
| userPoolId       | string | The ID of the user pool.                                                                                                   |
| userPoolName     | string | The Name of the user pool.                                                                                                 |
| userPoolMetadata | object | Configurations of the user pool.                                                                                             |
| appId            | string | The ID of the current user, **you can use appId to distinguish the source application of the user requirement**.                                                |
| appName          | string | The name of the current application.                                                                                            |
| appMetadata      | object | Configurations of the current application.                                                                                         |
| application      | string | The ID of the user pool.                                                                                                    |
| request          | object | The detailed information of current requirement, including: <br> `ip`: The IP of the client. <br> `geo`: The geographic location of the client which is parsed from the IP address. <br> `body`: The body of the requirement.  |

### The Rule of the Script's Return Value

#### The User Exists

When the user exists, you need to return user information to Approw, the format of user information can be found in document of [detailed fields of user profile](/guides/user/user-profile.md). For example:

```javascript
async function getUser(query, context) {
  // Implement your logic here
  return {
    id: 1, // must not empty
    email: "test@example.com",
    emailVerified: true,
    nickname: "Nick",
    photo: ""
  }
}
```

#### The User Does not Exist

When the user does not exist, you need to return `null`. Please do not throw an error.


#### Other Abnormal Errors

When user meets other errors, you can catch the error and return a friendly notice such as:

```javascript
async function getUser(query, context) {
  try {
    // Implement your logic here
  } catch (error) {
    throw new Error('Something went wrong ...')
  }
}
```

## Best Practice

### Provide Friendly Error Annoncements

When an unknown error occurs, we recommend throwing a standard `Error` object, Approw will catch this error and return it to the end user. For example, using `throw new Error("My nice error message")` and you will find this error log in the **History Log** of the customized database.

![](https://cdn.authing.cn/img/20210111163154.png)

### Disable the Database Connection When Exit the Function

Remeber to close the database connection after the whole script is run. You can use client.end() in the try/finally to make sure this command will be executed.

```javascript
try {
  const result = await client.query("YOUR QUERY");
} finally {
  // NOTE: always call `client.end()` here to close the connection to the database
  client.end();
}
```

### Example Functions

Assume we are using `postgres` as our database:

- You can use `env.DB_CONNECTION_URI` to get database connection string to create database connection.
- According to the query conditions in the `query` to generate query command(`query.id`, `query.email`, `query.username` and `query.phone`, these four parameters won't be empty at the same time).
- If user does not exist, return `null`.
- Finally return user information in valid format. The format of user information can be found in document of [detailed fields of user profile](/guides/user/user-profile.md).
- Call `try/finally` in `client.end()` to disable database connection.

```javascript
async function getUser(query, context) {
  // The first argument query contains following contents:
  // query.id
  // query.email
  // query.username
  // query.phone

  const { id, email, username, phone } = query;

  // The Second argument context contains information about the authentication context.
  // see http://core.authing.cn/connections/custom-db/config-custom-db-connection.html for more information.

  // This example uses the "pg" library
  // more info here: https://github.com/brianc/node-postgres
  const { Client } = require('pg');

  const client = new Client({
    connectionString: env.DB_CONNECTION_URI,
  });

  // Or you can:
  // const client = new Client({
  //   host: env.DB_HOST,
  //   port: env.DB_PORT,
  //   user: env.DB_USERNAME,
  //   password: env.DB_PASSWORD,
  //   database: env.DB_DATABASE,
  // });

  await client.connect();

  // 构建查询参数
  const queries = [];
  const parameters = [];
  let index = 1;
  if (id) {
    queries.push(`id = $${index}`);
    parameters.push(id);
    index += 1;
  }
  if (email) {
    queries.push(`email = $${index}`);
    parameters.push(email);
    index += 1;
  }
  if (phone) {
    queries.push(`phone = $${index}`);
    parameters.push(phone);
    index += 1;
  }
  if (username) {
    queries.push(`username = $${index}`);
    parameters.push(username);
    index += 1;
  }

  const QUERY = `SELECT * FROM users WHERE ${queries.join(' OR ')}`;

  try {
    const result = await client.query(QUERY, parameters);
    if (result.rows.length === 0) {
      return null;
    }
    const user = result.rows[0];
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      phone: user.phone,
      username: user.username,
      photo: user.photo,
      nickname: user.nickname,
      token: user.token,
      emailVerified: user.email_verified,
      phoneVerified: user.phone_verified,
      loginsCount: user.logins_count,
      lastIp: user.last_ip,
      gender: user.gender,
      address: user.address,
      company: user.company,
      birthdate: user.birthdate,
      website: user.website,
    };
  } catch (error) {
    throw new Error(`Execute query failed: ${error.message}`);
  } finally {
    // NOTE: always call `client.end()` here to close the connection to the database
    client.end();
  }
}
```