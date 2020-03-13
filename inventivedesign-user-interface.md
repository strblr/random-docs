Avant d'aborder cette doc, je vous conseille de prendre une heure pour lire et vous familiariser avec GraphQL, dans les grandes lignes au moins. Tout est sur [le site officiel](https://graphql.org/).

## Schéma

Voici le schéma GraphQL dans son intégralité :

```graphql
type User {
  id: ID!
  signupDate: String!
  online: Boolean!
  coworker: Boolean!
  groupCount: Int!
  invitationCount: Int!
  coworkerCount: Int!
  projectCount: Int!
  actionCount: Int!
  token: String!
  email: String! 
  name: String!
  job: String! 
  website: String! 
  bio: String! 
  avatar: String
  lastOnline: String!
}

type Query {
  user(id: ID!): User!
  userCount: Int!
  avatar(email: String!): String
  verifyUser(token: String!): User
}

type Mutation {
  signin(email: String!, password: String!): User!
}
```

Tous les champs ne seront évidemment pas exploitables dans vos applications (`coworker`, `projectCount`, etc. sont spécifiques à Finder / Physisolve).

## Opération

Pour exécuter une mutation (par exemple `signin`), GraphQL utilise la syntaxe suivante :

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

La réponse de cette requête est la suivante :

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

Si le mot de passe est mauvais (par exemple `tst` au lieu de `test`), la requête renverra une erreur :

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
            "path": [
                "signin"
            ]
        }
    ]
}
```

## Frameworks

Une requête GraphQL peut s'exprimer via un simple `POST` dans tous les langages / framework. C'est le corps de la requête qui va contenir le code GraphQL. Exemples :

#### NodeJS - Request

```javascript
var request = require('request');
var options = {
  'method': 'POST',
  'url': 'http://localhost:8084/finder/api',
  'headers': {
    'Content-Type': 'application/json'
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
request(options, function (error, response) { 
  if (error) throw new Error(error);
  console.log(response.body);
});
```

#### PHP - HTTP_Request2

```php
<?php
require_once 'HTTP/Request2.php';
$request = new HTTP_Request2();
$request->setUrl('http://localhost:8084/finder/api');
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


```graphql
```


```graphql
```


```graphql
```


```graphql
```
