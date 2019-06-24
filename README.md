# GraphQL App Recipes

This repo goes along with the tutorial on building AWS AppSync APIs using the Amplify GraphQL Transform library.

## Todo App

To deploy this app, use the following steps:

1. Create the Amplify project in your app

```sh
amplify init
```

2. Add the GraphQL API

```sh
amplify add api
```

Use the following schema

```graphql
type Todo @model {
  id: ID!
  name: String!
  description: String
}
```

## Whatsapp Clone

To deploy this app, use the following steps:

1. Create the Amplify project in your app

```sh
amplify init
```

2. Add authentication

```sh
amplify add auth
```

3. Add the GraphQL API

```sh
amplify add api
```

Use the following GraphQL schema:

```graphql
type User 
  @model 
  @auth(rules: [{ allow: owner, ownerField: "id", queries: null }]) {
  id: ID!
  username: String!
  avatar: S3Object
  conversations: [ConvoLink] @connection(name: "UserLinks")
  messages: [Message] @connection(name: "UserMessages")
  createdAt: String
  updatedAt: String
}

type S3Object {
  bucket: String!
  region: String!
  key: String!
}

type Conversation
  @model(
    mutations: { create: "createConvo" }
    queries: { get: "getConvo" }
    subscriptions: null
  )
  @auth(rules: [{ allow: owner, ownerField: "members" }]) {
  id: ID!
  messages: [Message] @connection(name: "ConvoMsgs", sortField: "createdAt")
  associated: [ConvoLink] @connection(name: "AssociatedLinks")
  name: String!
  members: [String!]!
  createdAt: String
  updatedAt: String
}

type Message 
  @model(subscriptions: null, queries: null) 
  @auth(rules: [{ allow: owner, ownerField: "authorId" }]) {
  id: ID!
  author: User @connection(name: "UserMessages", keyField: "authorId")
  authorId: String
  content: String!
  conversation: Conversation! @connection(name: "ConvoMsgs")
  messageConversationId: ID!
  createdAt: String
  updatedAt: String
}

type ConvoLink 
  @model(
    mutations: { create: "createConvoLink", update: "updateConvoLink" }
    queries: null
    subscriptions: null
  ) {
  id: ID!
  user: User! @connection(name: "UserLinks")
  convoLinkUserId: ID
  conversation: Conversation! @connection(name: "AssociatedLinks")
  convoLinkConversationId: ID!
  createdAt: String
  updatedAt: String
}

type Subscription {
  onCreateConvoLink(convoLinkUserId: ID!): ConvoLink
    @aws_subscribe(mutations: ["createConvoLink"])
  onCreateMessage(messageConversationId: ID!): Message
    @aws_subscribe(mutations: ["createMessage"])
}
```

4. Add Storage (S3)

```sh
amplify add storage

? Please select from one of the below mentioned services (Use arrow keys): Content (Images, audio, video, etc.)
? Please provide a friendly name for your resource that will be used to label this category in the project: imagestorage
? Please provide bucket name: <UNIQUE_BUCKET_NAME>
? Who should have access: Auth users only
? What kind of access do you want for Authenticated users? create/update, read, delete
```

5. Deploy the resources

```sh
amplify push
```
