# graphql-application
An example of a web application made with graphql

## Summary

- [ Introduction to GraphQL ](#introduction-to-graphql)
- [ Working with types ](#working-with-types)
- [ Querying data ](#querying-data)
- [ Querying batch data ](#querying-batch-data)
- [ Basic folder structure ](#basic-folder-structure)
- [ Changing data ](#changing-data)
- [ Using middlewares ](#using-middlewares)

<a name="introduction-to-graphql"></a>

## Introduction to GraphQL

The GraphQL comes from a new idea of how to manage data, because when using REST APIs we always have to set up a new endpoint to get information we need. In GraphQL, we work with the following idea:

1. We've got well defined data/schemas separated by nodes that we can access on demand (sometimes these nodes are parents of another nodes, so we can have them like a inner join on a database)

2. With one single request, we specify the node (and his children if we want to) that we wanna get the data from.

In order to work with this new kinda way to get data, when using GraphQL we have **TypeDefs** and **Resolvers**.

- Typedefs: We use them to describe the format of the data that exists on the nodes.

- Resolvers: All methods and business logics to get the needed data.

<a name="working-with-types"></a>

## Working with types

In GraphQL we have common types that we can use by default (Scalar Types) for some data response, such as: **Int**, **Float**, **String**, **Boolean**, **ID**.

Usually we define the schema type that way:
```graphql
type User {
	id: ID
	name: String!
	email: String!
	childrenName: [String]
}

# When using the exclamation after a scalar type (Ex: String!), we're saying that this field must be provided.
# When using the scalar type between brackets (Ex: [String]), we're specifying that's a array of the given scalar type.
```

If we want to add a custom scalar type, we need to do the following on our **TypeDefs**:
```graphql
scalar Date # Creates a new scalar of type Date
```

<a name="querying-data"></a>

## Querying data
Supposing that you've defined the following types:
```graphql
type Profile {
	id: Int
	type: String
}

type User {
	id: Int
	name: String!
	email: String!
	profile: Profile
}

type Query {
	users: [User]
}
```

The following resolver for **Query**:
```js
users() {
	return [
		{
			id: 1,
			name: "Guilherme",
			email: "guilhermebromonschenkel@gmail.com",
			profile_id: 1
		},
		{
			id: 2,
			name: "Mota",
			email: "mota@guilherr.me",
			profile_id: 2
		}
	]
}
```

And the following resolver for **User**:
```js
profile(user) {
	const profiles = [
		{
			id: 1,
			type: "common"
		},
		{
			id: 2,
			type: "admin"
		}
	]

	const profile = profiles.find(profile => profile.id === user.profile_id)

	return profile
}
```

When you get into the playground mode, you'll be able to query this data that way:
```graphql
query {
	users {
		id
		name
		email
		profile {
			id
			type
		}
	}
}
```

And you will receive:
```json
{
	"users": [
		{
			"id": 1,
			"name": "Guilherme",
			"email": "guilhermebromonschenkel@gmail.com",
			"profile": {
				"id": 1,
				"type": "common"
			}
		},
		{
			"id": 2,
			"name": "Mota",
			"email": "mota@guilherr.me",
			"profile": {
				"id": 2,
				"type": "admin"
			}
		}
	]
}
```

<a name="querying-batch-data"></a>

## Querying batch data

As shown above, when querying data we have to specify the data we need from the node (what means we can not query everything by default).

In order to make query of bunch data, we need to create **Fragments**.

**Fragments** are well defined bunch of data mapped by a name given by us in order to get them (it is like a view we commonly use on relational databases).

So, if we have the following type (supposing the resolver is already done):
```graphql
type User {
	id: Int
	name: String
	email: String
}
```

And I create the following fragment:
```graphql
fragment fullUserData on User {
	id
	name
	email
}
```

In the end, I'll be able to get all the fragment data by making the following query:
```graphql
{
	users {
		...fullUserData
	}
}
```

<a name="basic-folder-structure"></a>

## Basic folder structure
Below you're able to see the basic folder structuring for GraphQL projects. Usually the "index" file export all of the files in its same folder level.
```
src
│
└───schemas
│   │   index.graphql
│   │   User.graphql
│   │   Query.graphql
│   │   Mutation.graphql
│   ...
│
└───resolvers
│   │
│   └───Common
│   │   │   User.js
│   │   ...
│   │
│   └───Mutation
│   │   │   index.js
│   │   │   User.js
│   │   ...
│   │
│   └───Query
│   │   │   index.js
│   │   │   User.js
│   │   ...
│   │
│   └───Type
│   │   │   User.js
│   │   ...
│   │
│   │   index.js
|   ...
...
```

<a name="changing-data"></a>

## Changing data
Since we use the **query** method to fetch data, we can modify data by using the **mutation** method.

So, the same way we create a schema and resolver for **query**, we need to create a new one for **mutation**. The difference is that you will change data on the last one and fetch data on the other.

<a name="using-middlewares"></a>

## Using middlewares

Middlewares are commonly used on famous frameworks such as **ExpressJS**. They're really useful for managing business logic through requests (per example auth verification).

When dealing with GraphQL we can use middlewares as well but here they're called **Context**.

So, if I want to create a simple "console.log("hello world")" context to be executed before every query/mutation, I have to do the following:

```js
const context = (args) => {
	// args come with data such as 'req' and other stuffs

	console.log("Hello World")
}

const { ApolloServer } = require("apollo-server")

const server = new ApolloServer({
	typeDefs, // Schemas you defined previously
	resolvers, // Resolvers you defined previously
	context
})

server.listen().then(({ url }) => {
	console.log(`Running at ${url}`)
})
```