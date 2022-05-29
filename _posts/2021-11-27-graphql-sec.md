---
title: Fundamentals of GraphQL Security
date: 2021-11-27 12:00:00 +0500
categories: [Web Application Security]
tags: [api,graphql,web-application,security]
---

# Introduction

GraphQL is a query language for an API and not for a database i.e., it is database agnostic. Clients can use GraphQL to request for many types of data from multiple source in the API, as well as choose what parts of the data should be returned to the client.

## Schema

It is the blueprint of the GraphQL service that is used to define types and data used throughout the application and their requirements. Query and Mutations are two types in the schema definition. It also governs which data a client can request or mutate. To define types, GraphQL uses some primary types called *Scalar* Types, which are as follows &rarr;

1. Int &rarr; A signed 32‐bit integer.
2. Float &rarr; A signed double-precision floating-point value.
3. String &rarr; A UTF‐8 character sequence.
4. Boolean &rarr; true or false.
5. ID &rarr; The ID scalar type represents a unique identifier, often used to refetch an object or as the key for a cache. The ID type is serialized in the same way as a String, however, defining it as an ID signifies that it is not intended to be human-readable.
6. Custom Scalar types can be defined as follows &rarr; `scalar Date`.
7. Enums &rarr; A special type of scalar value which has a specific number of allowed values.

```json
enum PrivacyType {
  Public
  Private
}
```

## Query

All GraphQL services have a query type which is similar to an object type but is special because it is used to define the entry point for a query. A query can be sent as follows &rarr;

```json
{
  user {
    name
    avatarLink
  }
}
```

The query returns the following form of data &rarr;

```json
{
    'data': {
        'user': {
            'name': 'John Snow',
            'avatarLink': 'https://....'
         }       
    }
}
```

The query can be defined as follows &rarr;

```
type Query {
    user(id: ID!): User
    ...     # other query definition.
}
```

## Mutation

A mutation may or may not be defined for a GraphQL service. It can be used to modify the data store for the GraphQL service. It's like a union of the PUT, POST and DELETE request types. It can be defined in the following manner &rarr;

```
type Mutation {
    createUser(name: String!, avatarLink: String, dateOfBirth: String, location: String): User
    ... # Other mutations
}
```

A client can send a request with the following body &rarr;

```
mutation {
    createUser(name: 'Ned Stark') {
        name
	location
	avatarLink                               
    }
}
```

The response from the server could be something like the following &rarr;

```
{
    'data': {
        'createUser': {
            'name': 'Ned Stark',
            'location': null,
            'avatarLink': null,
         }
    }
}
```

GraphQL solves following 2 major problems of REST API implementation &rarr;

1. Unnecessary data fetching leads to utilizing high bandwidth (Over fetching)
2. Unnecessary implementation of different endpoints for different functions leads to complexity and need to do multiple calls from front end to get required data. (Under fetching)

## Resolver

A resolver is a function that’s responsible for populating the data for a single field in the schema, which could be by fetching data from the backend or via a third party API. The basic function is to convert operations into data.

# Security Concerns

The most basic security concern is that GraphQL does not provide authorizations checks for any actions to read or modify data through it.

GraphQL endpoints such as Playground and iGraphQL can be fuzzed for to find important information. A more complete list is as follows &rarr;

1. /graphql
2. /graphiql
3. /graphql/console
4. /graphql.php
5. /graphiql.php
6. /explorer
7. /altair
8. /playground
9. /graphql-playground
10. /graphql-explorer

GraphQL introspection query can be used to retrieve information such as queries and mutations from the schema. An introspection query for use through BurpSuite is as follows &rarr;

```
{"query":" query IntrospectionQuery {\n    __schema {\n      queryType { name }\n      mutationType { name }\n      types {\n        ...FullType\n      }\n      directives {\n        name\n        description\n        locations\n        args {\n          ...InputValue\n        }\n      }\n    }\n  }\n  fragment FullType on __Type {\n    kind\n    name\n    description\n    fields(includeDeprecated: true) {\n      name\n      description\n      args {\n        ...InputValue\n      }\n      type {\n        ...TypeRef\n      }\n      isDeprecated\n      deprecationReason\n    }\n    inputFields {\n      ...InputValue\n    }\n    interfaces {\n      ...TypeRef\n    }\n    enumValues(includeDeprecated: true) {\n      name\n      description\n      isDeprecated\n      deprecationReason\n    }\n    possibleTypes {\n      ...TypeRef\n    }\n  }\n  fragment InputValue on __InputValue {\n    name\n    description\n    type { ...TypeRef }\n    defaultValue\n  }\n  fragment TypeRef on __Type {\n    kind\n    name\n    ofType {\n      kind\n      name\n      ofType {\n        kind\n        name\n        ofType {\n          kind\n          name\n          ofType {\n            kind\n            name\n            ofType {\n              kind\n              name\n              ofType {\n                kind\n                name\n                ofType {\n                  kind\n                  name\n                }\n              }\n            }\n          }\n        }\n      }\n    }\n  }"}
```

The query part of the request without spaces whitespaces is as follows &rarr;

```
__schema{queryType{name},mutationType{name},types{kind,name,description,fields(includeDeprecated:true){name,description,args{name,description,type{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name}}}}}}}},defaultValue},type{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name}}}}}}}},isDeprecated,deprecationReason},inputFields{name,description,type{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name}}}}}}}},defaultValue},interfaces{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name}}}}}}}},enumValues(includeDeprecated:true){name,description,isDeprecated,deprecationReason,},possibleTypes{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name}}}}}}}}},directives{name,description,locations,args{name,description,type{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name,ofType{kind,name}}}}}}}},defaultValue}}}
```

The same query but URL encoded is as follows &rarr;

```
fragment+FullType+on+__Type+{++kind++name++description++fields(includeDeprecated%3a+true)+{++++name++++description++++args+{++++++...InputValue++++}++++type+{++++++...TypeRef++++}++++isDeprecated++++deprecationReason++}++inputFields+{++++...InputValue++}++interfaces+{++++...TypeRef++}++enumValues(includeDeprecated%3a+true)+{++++name++++description++++isDeprecated++++deprecationReason++}++possibleTypes+{++++...TypeRef++}}fragment+InputValue+on+__InputValue+{++name++description++type+{++++...TypeRef++}++defaultValue}fragment+TypeRef+on+__Type+{++kind++name++ofType+{++++kind++++name++++ofType+{++++++kind++++++name++++++ofType+{++++++++kind++++++++name++++++++ofType+{++++++++++kind++++++++++name++++++++++ofType+{++++++++++++kind++++++++++++name++++++++++++ofType+{++++++++++++++kind++++++++++++++name++++++++++++++ofType+{++++++++++++++++kind++++++++++++++++name++++++++++++++}++++++++++++}++++++++++}++++++++}++++++}++++}++}}query+IntrospectionQuery+{++__schema+{++++queryType+{++++++name++++}++++mutationType+{++++++name++++}++++types+{++++++...FullType++++}++++directives+{++++++name++++++description++++++locations++++++args+{++++++++...InputValue++++++}++++}++}}
```

The same query but with proper formatting (human readable) &rarr; 

```
 query IntrospectionQuery {
    __schema {
      queryType { name }
      mutationType { name }
      types {
        ...FullType
      }
      directives {
        name
        description
        locations
        args {
          ...InputValue
        }
      }
    }
  }
  fragment FullType on __Type {
    kind
    name
    description
    fields(includeDeprecated: true) {
      name
      description
      args {
        ...InputValue
      }
      type {
        ...TypeRef
      }
      isDeprecated
      deprecationReason
    }
    inputFields {
      ...InputValue
    }
    interfaces {
      ...TypeRef
    }
    enumValues(includeDeprecated: true) {
      name
      description
      isDeprecated
      deprecationReason
    }
    possibleTypes {
      ...TypeRef
    }
  }
  fragment InputValue on __InputValue {
    name
    description
    type { ...TypeRef }
    defaultValue
  }
  fragment TypeRef on __Type {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
                ofType {
                  kind
                  name
                }
              }
            }
          }
        }
      }
    }
  }
```

The result can be pasted into GraphQL voyager (in resources) to gain a graph based table visualization of queries available. The entire JSON object would need to be parsed to retrieve mutations. Otherwise, GraphiQL online can also be used to figure out mutations.

IDOR and AuthZ issues are common in GraphQL.

# Resources

1. [GraphQL Fundamentals](https://graphql.org/graphql-js/)
2. [GraphQL Voyager](https://apis.guru/graphql-voyager/)
3. [Hacking GraphQL for fun and profit - Part 1](https://infosecwriteups.com/hacking-graphql-for-fun-and-profit-part-1-understanding-graphql-basics-72bb3dd22efa)
4. [Hacking GraphQL for fun and profit - Part 2](https://infosecwriteups.com/hacking-graphql-for-fun-and-profit-part-2-methodology-and-examples-5992093bcc24)
5. [Exploit GraphQL endpoint](https://blog.yeswehack.com/yeswerhackers/how-exploit-graphql-endpoint-bug-bounty/)
