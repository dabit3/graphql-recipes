# GraphQL App Recipes

> This repo is an evolving work in progress. As I add more recipes, I will break these out into individual full-stack applications including the front end and adding additional examples. If you have any ideas, please feel free to either contribute or submit an issue!

![](header.jpg)

This repo goes along with the [tutorial on building AWS AppSync APIs using the Amplify GraphQL Transform library](https://dev.to/open-graphql/graphql-recipes-building-apis-with-graphql-transform-3jp0).

To learn more about GraphQL Transform library, check out [the documentation](https://aws-amplify.github.io/docs/cli-toolchain/graphql?sdk=js).

To learn more about building full stack serverless applications with GraphQL and AWS Amplify, check out my post [Infrastructure as Code in the Era of GraphQL and Full Stack Serverless](https://dev.to/dabit3/infrastructure-as-code-in-the-era-of-graphql-and-full-stack-serverless-11bc).

1. [Todo App](#todo-app)
2. [Events App](#event-app)
3. [Chat App](#chat-app)
3. [Multi-user Chat App](#multiuser-chat-app)
4. [E Commerce App](#e-commerce-app)
5. [WhatsApp Clone](#whatsapp-clone)
6. [Reddit Clone](#reddit-clone)
7. [Conference App](#conference-app)
8. [Instagram Clone](#instagram-clone)
9. [Giphy Clone](#giphy-clone)

> Some applications may require additional custom authorization logic for certain subscriptions that you may not want accessible to all users. To learn more, check out the documentation [here](https://docs.aws.amazon.com/appsync/latest/devguide/security-authorization-use-cases.html).

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

3. Deploy the resources

```sh
amplify push
```

## Event App

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

Use the following schema

```graphql
type Event @model
  @key(name: "itemType", fields: ["itemType", "time"], queryField: "eventsByDate")
  @auth(rules: [
    { allow: groups, groups: ["Admin"] },
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ]) {
    id: ID!
    name: String!
    description: String
    time: String!
    itemType: String!
    comments: [Comment] @connection #optional comments field
}

# Optional Comment type
type Comment @model
  @auth(rules: [
    { allow: owner, ownerField: "author" },
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ]) {
  id: ID!
  message: String!
  author: String
}
```

4. Deploy the resources

```sh
amplify push
```

## Chat app

To deploy this app, use the following steps:

1. Create the Amplify project in your app

```sh
amplify init
```

2. Add the GraphQL API

```sh
amplify add api
```

Use the following GraphQL Schema:

```graphql
type Conversation @model {
  id: ID!
  name: String
  messages: [Message] @connection(keyName: "messagesByConversationId", fields: ["id"])
  createdAt: String
  updatedAt: String
}

type Message
  @key(name: "messagesByConversationId", fields: ["conversationId"])
  @model(subscriptions: null, queries: null) {
  id: ID!
  conversationId: ID!
  content: String!
  conversation: Conversation @connection(fields: ["conversationId"])
  createdAt: String
}
```

## Multi-user Chat App

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

Use the following GraphQL Schema:

```graphql
type User
  @key(fields: ["userId"])
  @model(subscriptions: null)
  @auth(rules: [
    { allow: owner, ownerField: "userId" }
  ]) {
  userId: ID!
  conversations: [ConvoLink] @connection(keyName: "conversationsByUserId", fields: ["userId"])
  messages: [Message] @connection(keyName: "messagesByUserId", fields: ["userId"])
  createdAt: String
  updatedAt: String
}

type Conversation
  @model(subscriptions: null)
  @auth(rules: [{ allow: owner, ownerField: "members" }]) {
  id: ID!
  messages: [Message] @connection(keyName: "messagesByConversationId", fields: ["id"])
  associated: [ConvoLink] @connection(keyName: "convoLinksByConversationId", fields: ["id"])
  members: [String!]!
  createdAt: String
  updatedAt: String
}

type Message
  @key(name: "messagesByConversationId", fields: ["conversationId"])
  @key(name: "messagesByUserId", fields: ["userId"])
  @model(subscriptions: null, queries: null) {
  id: ID!
  userId: ID!
  conversationId: ID!
  author: User @connection(fields: ["userId"])
  content: String!
  conversation: Conversation @connection(fields: ["conversationId"])
  createdAt: String
  updatedAt: String
}

type ConvoLink
  @key(name: "convoLinksByConversationId", fields: ["conversationId"])
  @key(name: "conversationsByUserId", fields: ["userId"])
  @model(
    mutations: { create: "createConvoLink", update: "updateConvoLink" }
    queries: null
    subscriptions: null
  ) {
  id: ID!
  userId: ID!
  conversationId: ID!
  user: User @connection(fields: ["userId"])
  conversation: Conversation @connection(fields: ["conversationId"])
  createdAt: String
  updatedAt: String
}

type Subscription {
  onCreateConvoLink(userId: ID): ConvoLink
    @aws_subscribe(mutations: ["createConvoLink"])
  onCreateMessage(conversationId: ID): Message
    @aws_subscribe(mutations: ["createMessage"])
}
```

4. Deploy the resources

```sh
amplify push
```

## E-commerce App

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
# Products - Orders - Customers

type Customer @model(subscriptions: null)
  @auth(rules: [
    { allow: owner },
    { allow: groups, groups: ["Admin"] }
  ]) {
  id: ID!
  name: String!
  email: String!
  address: String
  orders: [Order] @connection(keyName: "byCustomerId", fields: ["id"])
}

type Product @model(subscriptions: null)
  @auth(rules: [
    { allow: groups, groups: ["Admin"] },
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ]) {
  id: ID!
  name: String!
  description: String
  price: Float!
  image: String
}

type Order @model(subscriptions: null)
  @key(name: "byCustomerId", fields: ["customerId", "createdAt"], queryField: "ordersByCustomerId")
  @auth(rules: [
   { allow: owner },
   { allow: groups, groups: ["Admin"] }
  ]) {
  id: ID!
  customerId: ID!
  total: Float
  subtotal: Float
  tax: Float
  createdAt: String!
  customer: Customer @connection(fields: ["customerId"])
  lineItems: [LineItem] @connection(keyName: "byOrderId", fields: ["id"])
}

type LineItem @model(subscriptions: null)
  @key(name: "byOrderId", fields: ["orderId"])
  @auth(rules: [
   { allow: owner },
   { allow: groups, groups: ["Admin"] }
  ]) {
  id: ID!
  orderId: ID!
  productId: ID!
  qty: Int
  order: Order @connection(fields: ["orderId"])
  product: Product @connection(fields: ["productId"])
  description: String
  price: Float
  total: Float
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
  @key(fields: ["userId"])
  @model(subscriptions: null)
  @auth(rules: [
    { allow: owner, ownerField: "userId" }
  ]) {
  userId: ID!
  avatar: String
  conversations: [ConvoLink] @connection(keyName: "conversationsByUserId", fields: ["userId"])
  messages: [Message] @connection(keyName: "messagesByUserId", fields: ["userId"])
  createdAt: String
  updatedAt: String
}

type Conversation
  @model(subscriptions: null)
  @auth(rules: [{ allow: owner, ownerField: "members" }]) {
  id: ID!
  messages: [Message] @connection(keyName: "messagesByConversationId", fields: ["id"])
  associated: [ConvoLink] @connection(keyName: "convoLinksByConversationId", fields: ["id"])
  members: [String!]!
  createdAt: String
  updatedAt: String
}

type Message
  @key(name: "messagesByConversationId", fields: ["conversationId"])
  @key(name: "messagesByUserId", fields: ["userId"])
  @model(subscriptions: null, queries: null) {
  id: ID!
  userId: ID!
  conversationId: ID!
  author: User @connection(fields: ["userId"])
  content: String!
  image: String
  conversation: Conversation @connection(fields: ["conversationId"])
  createdAt: String
  updatedAt: String
}

type ConvoLink
  @key(name: "convoLinksByConversationId", fields: ["conversationId"])
  @key(name: "conversationsByUserId", fields: ["userId"])
  @model(
    mutations: { create: "createConvoLink", update: "updateConvoLink" }
    queries: null
    subscriptions: null
  ) {
  id: ID!
  userId: ID!
  conversationId: ID!
  user: User @connection(fields: ["userId"])
  conversation: Conversation @connection(fields: ["conversationId"])
  createdAt: String
  updatedAt: String
}

type Subscription {
  onCreateConvoLink(userId: ID): ConvoLink
    @aws_subscribe(mutations: ["createConvoLink"])
  onCreateMessage(conversationId: ID): Message
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

## Reddit clone

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

Use the following GraphQL Schema:

```graphql
type User @model(subscriptions: null)
  @key(fields: ["userId"])
  @auth(rules: [
    { allow: owner, ownerField: "userId" }
  ]) {
  userId: ID!
  posts: [Post] @connection(keyName: "postByUser", fields: ["userId"])
  createdAt: String
  updatedAt: String
}

type Post @model
  @key(name: "postByUser", fields: ["authorId", "createdAt"])
  @auth(rules: [
    { allow: owner, ownerField: "authorId" },
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ]) {
  id: ID!
  authorId: ID!
  author: User @connection(fields: ["authorId"])
  postContent: String
  postImage: String
  comments: [Comment] @connection(keyName: "commentsByPostId", fields: ["id"])
  votes: [PostVote] @connection(keyName: "votesByPostId", fields: ["id"])
  createdAt: String
  voteCount: Int
}

type Comment @model
  @key(name: "commentsByPostId", fields: ["postId"])
  @auth(rules: [
    { allow: owner, ownerField: "authorId" },
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ]) {
  id: ID!
  authorId: ID!
  postId: ID!
  text: String!
  author: User @connection(fields: ["authorId"])
  votes: [CommentVote] @connection(keyName: "votesByCommentId", fields: ["id"])
  post: Post @connection(fields: ["postId"])
  voteCount: Int
}

type PostVote @model
  @auth(rules: [
    { allow: owner, ownerField: "userId"},
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ])
  @key(name: "votesByPostId", fields: ["postId"]) {
  id: ID!
  postId: ID!
  userId: ID!
  post: Post @connection(fields: ["postId"])
  createdAt: String!
  vote: VoteType
}

type CommentVote @model
  @auth(rules: [
    { allow: owner, ownerField: "userId"},
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ])
  @key(name: "votesByCommentId", fields: ["commentId"]) {
  id: ID!
  userId: ID!
  commentId: ID!
  comment: Comment @connection(fields: ["commentId"])
  createdAt: String!
  vote: VoteType
}

input VoteInput {
  type: VoteType!
  id: ID!
}

enum VoteType {
  up
  down
}
```

#### Custom resolver for votes

In the request mapping template,

```
# Set the vote ID as a combination of the postId and the user's userId.
#set($itemId = "$context.identity.username#$context.args.postId")
$util.qr($context.args.input.put("id", $util.defaultIfNull($ctx.args.input.id, $itemId)))

# delete or comment out the conditional expression code that does not allow the vote to be overridden:
#set( $condition = {
  "expression": "attribute_not_exists(#id)",
  "expressionNames": {
      "#id": "id"
  }
})
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

## Conference App

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

Use the following GraphQL Schema:

```graphql
type Talk @model
  @auth(rules: [
    { allow: groups, groups: ["Admin"] },
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ]) {
  id: ID!
  name: String!
  speakerName: String!
  speakerBio: String!
  time: String
  timeStamp: String
  date: String
  location: String
  summary: String!
  twitter: String
  github: String
  speakerAvatar: String
  comments: [Comment] @connection(keyName: "commentsByTalkId", fields: ["id"])
}

type Comment @model
  @key(name: "commentsByTalkId", fields: ["talkId"])
  @auth(rules: [
    { allow: owner, ownerField: "authorId" },
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ])
{
  id: ID!
  talkId: ID!
  talk: Talk @connection(fields: ["talkId"])
  message: String
  createdAt: String
  authorId: ID!
  deviceId: ID
}

type Report @model
  @auth(rules: [
    { allow: owner, operations: [create, update, delete] },
    { allow: groups, groups: ["Admin"] }
  ])
  {
  id: ID!
  commentId: ID!
  comment: String!
  talkTitle: String!
  deviceId: ID
}

type ModelCommentConnection {
  items: [Comment]
  nextToken: String
}

type Query {
  listCommentsByTalkId(talkId: ID!): ModelCommentConnection
}

type Subscription {
  onCreateCommentWithId(talkId: ID!): Comment
        @aws_subscribe(mutations: ["createComment"])
}
```

4. Deploy the resources

```sh
amplify push
```

## Instagram Clone

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

Use the following GraphQL Schema:

```graphql
type User @model(subscriptions: null)
  @key(fields: ["userId"])
  @auth(rules: [
    { allow: owner, ownerField: "userId" },
    { allow: private, operations: [read] }
    ]) {
  userId: ID!
  posts: [Post] @connection(keyName: "postsByUserId", fields: ["userId"])
  createdAt: String
  updatedAt: String
  following: [Following] @connection(keyName: "followingByUserId", fields: ["userId"])
}

type Post @model
  @key(name: "postsByUserId", fields: ["authorId"])
  @auth(rules: [
    { allow: owner ownerField: "authorId" },
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ]) {
  id: ID!
  authorId: ID!
  content: String!
  postImage: String
  author: User @connection(fields: ["authorId"])
  comments: [Comment] @connection(keyName: "commentsByPostId", fields: ["id"])
  likes: [PostLike] @connection(keyName: "postLikesByPostId", fields: ["id"])
}

type Comment @model
  @key(name: "commentsByPostId", fields: ["postId"])
  @auth(rules: [
    { allow: owner, ownerField: "authorId" },
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ]) {
  id: ID!
  postId: ID!
  authorId: ID!
  text: String!
  likes: [CommentLike] @connection(keyName: "commentLikesByCommentId", fields: ["id"])
  author: User @connection(fields: ["authorId"])
  post: Post @connection(fields: ["postId"])
}

type PostLike @model
  @auth(rules: [
    { allow: owner, ownerField: "userId" },
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ])
  @key(name: "postLikesByPostId", fields: ["postId"])
  @key(name: "postLikesByUser", fields: ["userId", "createdAt"], queryField: "likesByUser") {
  id: ID!
  postId: ID!
  userId: ID!
  user: User @connection(fields: ["userId"])
  post: Post @connection(fields: ["postId"])
  createdAt: String!
}

type CommentLike @model
  @auth(rules: [
    { allow: owner, ownerField: "userId" },
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ])
  @key(name: "commentLikesByCommentId", fields: ["commentId"])
  @key(name: "commentLikesByUser", fields: ["userId", "createdAt"], queryField: "likesByUser") {
  id: ID!
  userId: ID!
  postId: ID!
  commentId: ID!
  user: User @connection(fields: ["userId"])
  post: Post @connection(fields: ["postId"])
  createdAt: String!
}

type Following @model
  @auth(rules: [
    { allow: owner, ownerField: "followerId" },
    { allow: public, operations: [read] },
    { allow: private, operations: [read] }
  ])
  @key(name: "followingByUserId", fields: ["followerId"]) {
  id: ID
  followerId: ID!
  followingId: ID!
  follower: User @connection(fields: ["followerId"])
  following: User @connection(fields: ["followingId"])
  createdAt: String!
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

---

To learn more about uploading images in a GraphQL application with Amplify and AppSync, check out my tutorial [How to Manage Image & File Uploads & Downloads with AWS AppSync & AWS Amplify](https://dev.to/dabit3/graphql-tutorial-how-to-manage-image-file-uploads-downloads-with-aws-appsync-aws-amplify-hga)

## Giphy Clone

To deploy this app, use the following steps:

1. Create the Amplify project in your app

```sh
amplify init
```

2. Add a lambda function

```sh
amplify add function
```

3. Enter the following when prompted
   ![lambdafunction](screenshots/giphy-clone/create-lambda-function.png)

4. From the root of your project, change into your function's source directory

```sh
cd amplify/backend/function/giphyfunction/src
```

5. Install [axios](https://www.npmjs.com/package/axios) and [dotenv](https://www.npmjs.com/package/dotenv)

```sh
npm install axios && npm install -D dotenv
```

6. Update the contents of your functions `index.js` file

```js
const axios = require("axios");
const dotenv = require("dotenv");
dotenv.config();

exports.handler = (event, _, callback) => {
  const { GIPHY_API_KEY } = process.env;
  let apiUrl = `https://api.giphy.com/v1/gifs/search?api_key=${GIPHY_API_KEY}&q=hello`;

  if (event.arguments) {
    const { searchTerm = "hi", limit = 25 } = event.arguments;
    apiUrl = `https://api.giphy.com/v1/gifs/search?api_key=${GIPHY_API_KEY}&q=${searchTerm}&limit=${limit}`;
  }

  axios
    .get(apiUrl)
    .then(response => callback(null, response.data.data))
    .catch(err => callback(err));
};
```

7. Login to [Giphy's Developer portal](https://developers.giphy.com/dashboard/) and create an app to get an API Key.

8. Back in the function's `src` directory, create a `.env` file and add in the API Key that was obtained in the previous step as a secret.

```sh
GIPHY_API_KEY=<YOUR_API_KEY>
```

**Your .env file should NOT be committed to Github**

9. Head back to the root of your project

```sh
cd ../../../../../
```

10. Add the GraphQL API

```sh
amplify add api
```

Use the following GraphQL Schema:

```graphql
type Gif {
  id: ID!
  slug: String!
  images: GifImage!
}

type GifImage {
  original: GifAttributes!
  fixed_width: GifAttributes!
}

type GifAttributes {
  url: String!
  width: String!
  height: String!
}

type Query {
  getGifs(searchTerm: String, limit: Int): [Gif]
    @function(name: "giphyfunction-${env}")
}
```

11. Deploy the Resources

```sh
amplify push
```
