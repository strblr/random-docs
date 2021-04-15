# User doc - Inventive Design

Before reading this doc, I suggest to familiarize yourself a bit with GraphQL. Everything's on [their official website](https://graphql.org/).

## Schema

Here the part of the GraphQL schema that's of interest when handling users :

```graphql
# This type represents a user :

type User {
  id: ID! # unique id
  signupDate: String!
  online: Boolean!
  coworker: Boolean! # is the user a corworker of the user making the request ?
  coworkerCount: Int! # total number of coworkers
  sessionCount: Int! # total number of sessions the user's involved in
  teamCount: Int! # total number of teams the user's involved in
  projectCount: Int! # total number of projects the user's involved in
  invitationCount: Int! # total number of pending invitations for that user
  token: String! # auth token ; /!\ this is only available on User returned by verifyUser and signin /!\
  email: String!
  state: UserState! # see "enum UserState"
  status: UserStatus! # see "enum UserStatus"
  name: String!
  job: String!
  organizations: [Organization!]!
  website: String!
  skills: [Skill!]! # see "type Skill"
  bio: String!
  avatar: String
  lastOnline: String!
}

# This type is used to return paginated lists of users :

type UserPage {
  total: Int!
  items: [User!]!
}

# This enum identifies the account state of the user :

enum UserState {
  INVITED
  UNCONFIRMED
  ACTIVE
  BANNED
  DEACTIVATED
}

# This enum identifies the role of the user :

enum UserStatus {
  ADMIN
  MANAGER
  MEMBER
}

type Skill {
  name: String!
  rate: Int!
}

# Here are all user-related queries :

type Query {
  # Get user by id :
  user(id: ID!): User!

  # Get paginated lists of users :
  users(
    organizations: [ID!] # filter by organizations
    confirmed: Boolean # filter by confirmation state
    online: Boolean # filter by current presence
    search: String # filter by text
    sort: String # you can order the list according to a specific field
    sortOrder: Int # you can precise the direction of that ordering
    limit: Int! # how many users per page ?
    page: Int! # what page ?
  ): UserPage!

  # Get the avatar of a user given the email :
  avatar(email: String!): String

  # Decode an auth token :
  verifyUser(token: String!): User
}

# Here are all the relevant user-related mutations :

type Mutation {
  # The signin mutation (if successful, returns a User with a "token" field) :
  signin(email: String!, password: String!): User!

  # Invite users by email (rejected if not admin or manager) :
  inviteUsers(emails: [String!]!, organizations: [ID!]!): [User!]!

  # Reinvite a user by id (rejected if not admin or manager) :
  reinviteUser(id: ID!): User!

  # Send a password recovery email :
  recoverPassword(email: String!): Boolean!

  # Confirm the signup of a user (rejected if not admin) :
  confirmUser(id: ID!): User!

  # Edit a user's profile :
  editUser(
    id: ID!
    email: String!
    status: UserStatus!
    oldPassword: String!
    password: String!
    name: String!
    job: String!
    organizations: [ID!]!
    website: String!
    skills: [SkillInput!]!
    bio: String!
    avatar: String
  ): User!

  # Ban a user (rejected if not admin) :
  banUser(id: ID!): User!

  # Deactive a user's account (rejected if not done by the targeted user) :
  deactivateUser(id: ID!): User!
}
```

- Signups have to happen on the inventivedesign ecosystem apps

## Opération

To execute a mutation (for example "signin"), GraphQL uses the following syntax :

```graphql
mutation {
  signin(email: "test@test.fr", password: "test") {
    id
    online
    name
    job
    website
    token
  }
}
```

If successful, the response will look like :

```json
{
  "data": {
    "signin": {
      "id": "5cd0f843acf6e83cc265083b",
      "online": false,
      "name": "Elisabeth Ruecker",
      "job": "Software Developer",
      "website": "https://www.drodd.com/funny-team-names/science-team-names.html",
      "token": "................"
    }
  }
}
```

If there's an error, for example a bad password, the request will return the following error :

```json
{
  "data": null,
  "errors": [
    {
      "message": "The password is incorrect",
      "locations": [
        {
          "line": 2,
          "column": 13
        }
      ],
      "path": ["signin"]
    }
  ]
}
```

## Frameworks

A GraphQL request can be expressed via a simple "POST" http request in all languages / frameworks. The body of the request will contain the actual GraphQL code. Examples :

#### NodeJS - Request

```javascript
var request = require("request");
var options = {
  method: "POST",
  url: "monUrl",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    query: `
        mutation {
            signin(email: "test@test.fr", password: "test") {
                id
                online
                name
                job
                website
                token
            }
        }
    `,
    variables: {}
  })
};
request(options, function(error, response) {
  if (error) throw new Error(error);
  console.log(response.body);
});
```

#### PHP - HTTP_Request2

```php
<?php
require_once 'HTTP/Request2.php';
$request = new HTTP_Request2();
$request->setUrl('monUrl');
$request->setMethod(HTTP_Request2::METHOD_POST);
$request->setConfig(array(
  'follow_redirects' => TRUE
));
$request->setHeader(array(
  'Content-Type' => 'application/json'
));
$request->setBody('{"query":"mutation {\\n    signin(email: \\"test@test.fr\\", password: \\"test\\") {\\n        id\\n        online\\n        name\\n        job\\n        website\\n        token\\n    }\\n}\\n","variables":{}}');
try {
  $response = $request->send();
  if ($response->getStatus() == 200) {
    echo $response->getBody();
  }
  else {
    echo 'Unexpected HTTP status: ' . $response->getStatus() . ' ' .
    $response->getReasonPhrase();
  }
}
catch(HTTP_Request2_Exception $e) {
  echo 'Error: ' . $e->getMessage();
}
```

#### PHP - cURL

```php
<?php
  $url      = "monUrl";
  $email    = "test@test.fr";
  $password = "test";
  
  $query    = <<< "GRAPHQL"
      mutation {
          signin(email: "$email", password: "$password") {
              id
              online
              name
              job
              website
          }
      }
  GRAPHQL;
  $header = array(
    'Content-Type: application/json'
  );
  
  $result = request($url, $query, $header);
  echo $result; // Only to show the server's response.

  function request($url, $query, $header = [], $variables = []) {
    $curl = curl_init();
    
    curl_setopt_array($curl, array(
      CURLOPT_URL            => $url,
      CURLOPT_RETURNTRANSFER => true,
      CURLOPT_ENCODING       => '',
      CURLOPT_MAXREDIRS      => 10,
      CURLOPT_TIMEOUT        => 0,
      CURLOPT_FOLLOWLOCATION => true,
      CURLOPT_HTTP_VERSION   => CURL_HTTP_VERSION_1_1,
      CURLOPT_CUSTOMREQUEST  => 'POST',
      CURLOPT_POSTFIELDS     => json_encode(array(
          'query'     => $query,
          'variables' => $variables
      )),
      CURLOPT_HTTPHEADER     => $header
    ));

    $response = curl_exec($curl);
    curl_close($curl);

    return $response; 
  }
  ```
