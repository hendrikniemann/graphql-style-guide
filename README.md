# GraphQL Schema Style Guide

This schema style guide summarizes how we design the GraphQL schema at _Kiron Open Higher Education_. These principles are derived from our experience of running a GraphQL API over the last two years and inspired by the previous work of various people in the community.

This guide starts at a very high level and then tries to be more specific in how to achieve the overarching principles in the implementation.

## Table of contents

<!-- toc -->

- [Philosophy](#philosophy)
  * [Precise types](#precise-types)
  * [High level APIs](#high-level-apis)
  * [Leveraging the type system](#leveraging-the-type-system)
  * [Strict conventions](#strict-conventions)
- [Design best practices](#design-best-practices)
  * [Unique id fields](#unique-id-fields)
  * [Non nullable by default](#non-nullable-by-default)
  * [List are not nullable](#list-are-not-nullable)
- [Types](#types)
  * [Casing](#casing)
  * [Singular / Plural](#singular--plural)
  * [Output types](#output-types)
  * [Input types](#input-types)
- [Fields](#fields)
  * [Casing](#casing-1)
  * [Singular / Plural](#singular--plural-1)
  * [Naming](#naming)
- [Arguments](#arguments)
  * [Casing](#casing-2)
- [Mutations](#mutations)
  * [Naming](#naming-1)
- [Queries](#queries)
- [Naming](#naming-2)
  * [Collection queries](#collection-queries)
  * [By id queries](#by-id-queries)
  * [Viewer queries](#viewer-queries)
- [Mutations](#mutations-1)
  * [Create Mutations](#create-mutations)
  * [Delete Mutations](#delete-mutations)
  * [Mutation results](#mutation-results)

<!-- tocstop -->

## Philosophy

At Kiron we use GraphQL to create flexible interfaces that enable us to build better products. Our software design evolves around the GraphQL schema. We want to build APIs that are intuitive to use and the API consumer's user experience is our highest value.

To achieve this we apply the following overarching principles:

### Precise types

While creating a flexible type model increases the type reusability and can lead to smaller type models we should instead aim for precise types. Exposed types should be tailored to the data models underneath and reduce the amounts of nullable fields. This helps the consumer of the API to know exactly what to expect. GraphQL features such as interfaces and type unions help us with that goal. Creating new types is cheap (compared to REST) and scalable. Therefore we can create stronger abstractions than our underlying SQL databases or REST resources. We are encouraged to derive from the underlying models to improve the user experience.

### High level APIs

Our API is more than just an database access layer. Very little business logic should be handled by to the frontend because we cannot assume that the client knows the underlying rules. If the backend _can_ handle certain calculation or business logic it _should_ do so. In GraphQL fields are only calculated/resolved if they are demanded by the frontend. This allows us to create additional computed fields in our types without polluting the data model.

_Example_: A course is full when the number of participants reached the maximum amount of participants

```graphql
type Course {
  # bad
  participants: Int!
  maxParticipants: Int!

  # good
  full: Boolean!
}
```

### Leveraging the type system

GraphQL makes use of a strict type system in the Schema Definition Languages (SDL) as well as in the query languages (GraphQL). In GraphQL every property or argument not only carries a name but also a type and a description, we can specify our API without creating external documentation, guides or naming conventions.

### Strict conventions

The API should be consistent in the way it is designed and shaped. This style guide will cover many rules to make the API consistent. Sometimes we have to go even further to deliver consistency on a higher level. Consistency might often be a tradeoff for another principle and the team is responsible to make decisions on when to favour consistency over other principles.

## Design best practices

### Unique id fields

Every output object type that represents an entity should have a single identifier field that uniquely identifies the resource within the type. The identifier field is named `id` and of type `ID!`. The identifier field is owned by the API and while its value should not change over time, the possibility of change should be assumed. The `ID` scalar is serialised as string in GraphQL responses and has to be handled as as such in the frontend code.

### Non nullable by default

Generally types should only be nullable if this is a necessity of the domain (1 to 0..1 relationships). In contrast many companies prefer to make as little guarantees as possible ([the hidden cost of non-nullable fields](https://medium.com/@calebmer/when-to-use-graphql-non-null-fields-4059337f6fc8)). In our (fairly small) API we have found that the only places where non-null guarantees are hard to make are 1 to 1 relationships.

### List are not nullable

List usually are not nullable. Instead a missing value is represented by the empty list. Lists should not contain null values since they carry no meaning on the type level.

_Example:_

```graphql
type Person {
  fiends: [Person]!
}
```

If the `friends` field returns `[null]` the meaning of the `null` value is not obvious. One possible interpretation is that the friend represented by the null value is not known to the user. This is implicit knowledge that does not directly follow from the type system. There are many more explicit solutions like the introduction of a `PersonFriendConnection` type that contains the `viewerShownFriends` and the `viewerHiddenFriends`.

## Types

### Casing

Type names are in pascal case. That means they start with a capital letter. Type names should not contain whitespace dashes or underscores.

| Examples    |                                                      |
| :---------- | :--------------------------------------------------- |
| Good        | `User`, `SocialSecurityNumber`, `UpdateUserResult`   |
| Not allowed | `user`, `social_security_number`, `updateUserResult` |

### Singular / Plural

Type names should always be singular. In GraphQL collections are expressed using lists, therefore a type should not be plural.

| Examples    |                              |
| :---------- | :--------------------------- |
| Good        | `User`, `UpdateUserResult`   |
| Not allowed | `Users`, `UpdateUserResults` |

### Output types

Output types should be named according to the style rules and their name in the system. That means that type names should be consistent within the stack and with its usage in the organisation.

Types should not have their kind name (e.g. enum type or scalar type) in their name. If a type is an object type, enum type or scalar is already defined by the type system and does not have to be repeated in the name itself.

| Examples    |                                                   |
| :---------- | :------------------------------------------------ |
| Good        | `Gender`, `CourseStatus`, `Date`                   |
| Not allowed | `GenderEnumType`, `CourseStatusEnum`, `DateScalar` |

### Input types

_TODO_

~Input object types should be postfixed with `Input`. Input types often behave in different ways than~

| Examples    |                                   |
| :---------- | :-------------------------------- |
| Good        | `updateUser` -> `UpdateUserInput` |
| Not allowed | `updateUser` -> `UserUpdate`      |

## Fields

### Casing

Field names should be in camel case. That means they start with a capital letter. Field names should not contain whitespace, dashes or underscores.

| Examples    |                                                         |
| :---------- | :------------------------------------------------------ |
| Good        | `user`, `socialSecurityNumber`, `updateProfile`   |
| Not allowed | `User`, `social_security_number`, `updateprofile` |

### Singular / Plural

Fields that return single (non list type) values should be singular, fields that return list types should be plural.

| Examples    |                                                                  |
| :---------- | :--------------------------------------------------------------- |
| Good        | `users: [User!]!`, `socialSecurityNumber: SocialSecurityNumber!` |
| Not allowed | `user: [User!]!`, `socialSecurityNumbers: SocialSecurityNumber!` |

### Naming

Fields should be as short as possible and should neither contain the type name of the object type they are defined on nor their output type (sometimes there is a grey zone when the type name is part of the name, e.g. `dateOfBirth`).

| Examples    |                            |
| :---------- | :------------------------- |
| Good        | `name`, `birthday`         |
| Not allowed | `fileName`, `birthdayDate` |

_Debatable and currently not done at all in our API. In GraphQL you can see the type of a variable easily in the schema. This goes a bit against the other rules outlined above..._

~Boolean fields should be prefixed with `is` or `has` (or similar) to create a more speakable field name. This is generally a good practice in most programming languages or database designs and should therefore be aligning well with the other parts of the codebase.~

---

## Arguments

### Casing

Arguments names should be in camel case. They are in that sense very similar to fields.

| Examples    |                                  |
| :---------- | :------------------------------- |
| Good        | `id`, `update`, `securityNumber` |
| Not allowed | `ID`, `securityNumber`           |

## Mutations

For mutations all the same rules apply as for normal field types and the following naming conventions on top.

### Naming

Mutation names should be composed of a verb that expresses the type of action happening and the object name the mutation is operating on.

| Examples    |                                                     |
| :---------- | :-------------------------------------------------- |
| Good        | `updateProfile`, `setCourseStatus`, `trackPageView` |
| Not allowed | `profile`, `setStatus`, `track`                     |

---

## Queries

For queries all the same rules apply as for normal field types and the following naming conventions on top.

## Naming

### Collection queries

Collection queries are queries that return a collection of (all) values of a type. The query field name should be the plural of the type name. In the example below the `countries` query returns all values of the `Country` type. The return value is a non-nullable list of non-nullable values.

_Example_: The `countries` query returns all countries in the database.

```graphql
type Query {
  countries: [Country!]!
}
```

Collection queries can have a filter applied to them using an argument called `filter` of specific input type created for the field composed of the typename and the postfix `Filter`.

_Example_: To get all countries in Europe the countries field accepts a filter:

```graphql
enum Continent {
  ANTARCTICA
  AFRICA
  ASIA
  EUROPE
  NORTH_AMERICA
  OCEANIA
  SOUTH_AMERIC
}

input CountryFilter {
  continent: Continent
}

type Query {
  countries(filter: CountryFilter): [Country!]!
}
```

If a collection query cannot return any matching results it returns an empty array. It does not return null (not allowed by the schema) nor does it error.

### By id queries

By id queries are queries that query for a single element with a given id. The query name should be the singular of the type name (usually the type name itself since type names must be singular). The return value is nullable and null is returned when a value with the specified id is not found. The field **does not error** (by throwing a exception in the resolver) when the id cannot be located in the database.

_Example_: With the `country` field a single country can be selected by id

```graphql
type Query {
  country(id: ID!): Country
}
```

### Viewer queries

Viewer queries are similar to the collection and by id queries but already have a filter parameter applied: The user id / account id of the viewer. Viewer queries are located inside of the `viewer` namespace.

_Example_: The `courses` root query returns all courses. The `viewer.courses` field returns all courses that belong to the current viewer.

```graphql
type Query {
  viewer: Viewer!
  courses: [Course!]!
}

type Viewer {
  courses: [Course!]!
}
```

## Mutations

Mutations are in a sense very special because coming up with a good mutation design is one of the hardest parts when building a GraphQL schema. At Kiron we follow the approach outlined by Oleg Ilyenko in a [blog post on mutation] design(https://techblog.commercetools.com/modeling-graphql-mutations-52d4369f73b1), which practically applies the work of many other community members.

We design three types of mutations to cover all use cases. The types follow strong naming conventions. In the classic [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) sense our queries are used for reading the data. What we are left with are _create_, _update_ and _delete_ operations for our data.

With our mutations we also follow our design principles mentioned above. The API should only offer mutations that are needed and save. This is not only a good practice to secure the API but also keeps the interface simple.

### Create Mutations

Create mutations are mutations that create a new entity in the database. The mutation is named after the type of entity it is creating. The mutations must be named in the following schema `create<Entity Name>`. When you want to create a new `User` the mutation to use is the `createUser` mutation.

Create mutations usually take a single required argument `draft` of the draft type of the entity `<Entity Name>Draft`. The draft contains all the arguments that are used to create the instance. This allows us to make all arguments mandatory that are essential for the type and other fields nullable, that contain optional data for the creation. Mutations can have some implicit parameters for example the viewer or the current date.

Examples:

```graphql
type Mutation {
  # good
  createUser(draft: UserDraft!): CreateUserResult!
  createCertificate(draft: CertificateDraft!): CreateCertificateResult!

  # bad
  newUser(user: UserInput!): NewUserResult!
}

input CertificateDraft {
  title: String!
  score: Float
}

# Certificate draft would be enough to create Certificate type with fields
type Certificate {
  id: ID!
  title: String!
  score: Float
  createdAt: Date!
  updatedAt: Date!
}
```

### Delete Mutations

Delete mutations are used to delete a single entity from the database. The mutation is named after the type of entity it is deleting. The mutations must be named in the following schema `delete<Entity Name>`. When a mutation deletes a `User` the mutation is named `deleteUser`.

### Mutation results

Mutation results should contain the name of the mutation and should be postfixed with `Result`. The mutation result is always non-nullable.

| Examples    |                                    |
| :---------- | :--------------------------------- |
| Good        | `updateUser` -> `UpdateUserResult` |
| Not allowed | `updateUser` -> `UserUpdate`       |

Mutation results have two standard fields that need to be present in every result object type:

The `success` field indicates the status of the request and is either `true` if the mutation succeeded or `false` if it failed. The `errors` field contains a list of user errors that occurred during the execution of the mutation. This allows us to differentiate between the errors that happen due to bugs or problems in the implementation and user errors like failed validations. While it is hard to draw a clear line the rule is that if the error might address the enduser it should be inside of the error field.

Mutation results should also always contain the updated or newly created entity usually in a field that is named after the type name.

_Example:_

```graphql
type Mutation {
  createUser(draft: UserDraft): CreateUserResult!
}

type CreateUserResult {
  success: Boolean!
  errors: [Error!]!
  user: User
}
```
