---
title: Graph API
id: graph-api
---
import WorkInProgress from '../components/WorkInProgress.mdx'

# Graph API

Graph API provides a graphql based interface for storing and querying structured graph. Decentralized application
developers for Box can use this API to create, update and delete JSON documents using a standard graphql interface
directly on their Box. The Graph API is a part
of [Fula Client](https://github.com/functionland/fula/tree/main/libraries/fula-client) and you can invoke it using
the `graphql` interface.

`async graphql(query: string, variableValues?: never, operationName?: string)`

Arguments are:

- `query`: The graphql query string, see below for more information
- `variableValues?`: A JSON object specifying values for variable you used in the query string. You can specify
  variables in the query string using `$` and then provide values for them in `variableValues` when you want to execute
  the query or mutation. You can find an example on this page.
- `operationName?`: An optional name for your operation. _This argument is unused right now_

The method resolves to the result of the operation.

Currently, there are 4 types of mutation and a single query type that you can use. In this document you can find the
definition and a simple example for each operation, if you are familiar with graphql schemas, you may find the
current [graphql schema](https://github.com/functionland/fula/blob/main/apps/box/src/graph/gql-engine/schema.ts)  for
the Graph API useful.

## Queries

Every query operation takes a `collection` argument that refers to the name for collection you want to query. If a
collection with this name does not exist, the operation simply returns an empty output.

### read

Fetches a previously stored document based on a filter object.

`read (input:ReadInput): [JSON]`

The `input` argument should contain:

- `collection:String` (required): name of the collection.
- `filter: JSON`: a [Filter object](#filter-objects) that determines each document's existence in the output.

#### Example

This query operation finds all `profile` documents that has an `age` field greater than `20` and then returns
their `id, name, age`.

```javascript
const readQuery = `
  query {
    read(input:{
      collection:"profile",
      filter:{
        age: {gt: 50}
      }
    }){
      id
      name
      age
    }
  } 
`;

const res = await client.graphql(readQuery)
```

## Mutations

Every mutation operation takes a `collection` argument that refers to the name for collection you want to mutate. A new
collection will get created if the collection name does not exist.

### create

Creates a new document in a collection and returns the created document.

`create (input:CreateInput!): JSON`

The `input` argument should contain:

- `collection: String!` (required): name of the collection
- `values: JSON!`: document to be created

#### Example

This mutation creates a new document in the `profile` collection.

__*Note*__: You can use variable values in the query or mutation operation. `variables` argument does it for you:

```javascript
export const createMutation = `
  mutation addProfile($values:JSON){
    create(input:{
      collection:"profile",
      values: $values
    }){
      id
      name
      isActive
    }
  }
`;

const res = client.graphql(createMutation, {
    values: [{
        id: 1,
        name: 'Mehdi',
        isActive: false
    }]
}, 'CreateProfile')
```

### update

Updates a document. Note that the `values` argument determines the filter and the update at the same time.
`update (input:UpdateInput!): [JSON]`

The `input` argument contains:

- `collection: String!` (required): the collection name
- `values: JSON!`: document to be updated. This is used for updating the document also.

#### Example

This mutation finds a document with the `id` field and updates its `name`.

```javascript
export const updateMutation = `
  mutation updateProfile($values:JSON){
    update(input:{
      collection:"profile",
      values: $values
    }){
      id
      name
      age
    }
  }
`;

const res = await graphql(updateMutation, {
    values: [{
        id: 1,
        name: 'Mehdi',
    }]
}, 'UpdateProfile')
```

### delete

Deletes a document based on `id` field.

__*Note*__: Since the box uses orbitdb as the underlying graphbase, the `id` field is reserved by the db, if you don't
specify an `id` argument in the creation time of a document, an auto-generated one will be used.

`delete (input:DeleteInput!):[ID!]`

The `input` argument contains:

- `collection: String!` (required): the collection name
- `ids: [ID!]`: the `id`s for documents to be deleted

#### Example

```javascript
export const deleteMutation = `
  mutation deleteProfile($values:JSON){
    delete(input:{
      collection:"profile",
      ids: $values
    })
  }
`;
const res = await client.graphql(deleteMutation, {values: ['1', '2']}, 'DeleteProfile')
```

## Filter Objects

Filter objects are used to choose a subset of the documents in a collection depending on their attributes. They are
intended to be like [MongoDB Queries](https://docs.mongodb.com/manual/tutorial/query-documents/).

The graphql engine traverses the filter object recursively until it reaches an Atomic Filter Object. In addition, ATOs
can get combined and make a more complex filter object by Logical Operators.

### Atomic Filter Object

The simplest form of a filter object that describes an expected value for an attribute. This expectation can be
expressed by an Operator. For example this ATO expects the `age` attribute for a document to be greater than `15`

```javascript
{
    age: {
        gt: 15
    }
}
```

Every key in an ATO refers to an attribute name and the value for that key is the expectation expression. In the
expression you can use value operators (listed below) to define the criteria.

### Value Operators

Value Operators can be used to define a criteria for an attribute.

__*Note*__: Value Operator names are reserved by the `graphql-engine` and you can't use them as attribute names.

- `eq` (Equal to)
- `ne` (Not equal to)
- `gt` (Greater than)
- `gte` (Greater than or equal)
- `lt` (Lower than)
- `lte` (Lower than or equal)
- `in` (Be in [array])
- `nin` (Not be in [array])

### Logical Operators

To make more complex filter objects you can combine ATOs with logical operators `or`, `and`. Each of these operators
takes an array of filters.

#### Example

This example demonstrate the usage of filters.

```javascript
filter: {
    and: [
        {name: {nin: ["keyvan", "mahdi"]}},
        {
            or: [
                {age: {gt: 45}},
                {age: {lt: 15}}
            ]
        }
    ]
}
```

## Subscription
FULA client's Graph API provides subscription for queries. You can subscribe to a query's result and get the new result on each change. To do so you can use `graphqlSubscription` method.

`async function* graphqlSubscribe(query: string, variableValues?: never, operationName?: string)`

The interface is almost identical to `graphql` method except that `graphqlSubscribe` returns an `AsyncIterable`

Arguments are:
- `query`: The graphql query string, see below for more information
- `variableValues?`: A JSON object specifying values for variable you used in the query string. You can specify
  variables in the query string using `$` and then provide values for them in `variableValues` when you want to execute
  the query or mutation. You can find an example on this page.
- `operationName?`: An optional name for your operation. _This argument is unused right now_

The method returns an [`AsyncIterable`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of) that generates a new result based on query filters every time the collection is changed.

#### Example
```javascript
const readQuery = `
  query {
    read(input:{
      collection:"profile",
      filter:{
        age: {gt: 50}
      }
    }){
      id
      name
      age
    }
  } 
`;

const resultIterator = client.graphqlSubscribe(readQuery)

for await (const res of resultIterator){
    console.log(res)
}

```
<WorkInProgress />
