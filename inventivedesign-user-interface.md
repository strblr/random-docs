Schema :

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
