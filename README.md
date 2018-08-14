# GraphQL Schema Style Guide

This schema style guide summarizes how we design the GraphQL schema at _Kiron Open Higher Education_. These principles are derived from our experience of running a GraphQL API for our single page web app over the last two years and and inspired by various people in the community.

## Philosophy

At Kiron we use GraphQL to create flexible interfaces that enable us to build better products. We want to build APIs that are intuitive to use and the consumer's user experience is our highest property.

To achieve this we apply the following overarching principles:

### Precise types

While creating a flexible type model increases the type reusability and can lead to smaller type models we should instead aim for precise types. Exposed types should be tailored to the data models underneath and reduce the amounts of nullable fields. This helps the consumer of the API to know exactly what to expect. GraphQL features such as interfaces and type unions help us with that goal. Creating new types is cheap (compared to REST) and scalable. Therefore we can create stronger abstractions than our underlying SQL databases or REST resources. We are encouraged to derive from the underlying models to improve the user experience.

### Strong abstractions

Our API is more than just an database access provider. If the backend _can_ handle certain calculation or business logic it _should_ do so.

We think that very little business logic should leak to the frontend. In GraphQL fields are only calculated/resolved if they are demanded by the frontend. This way we can create additional computed fields in our types without polluting the data model.

_Example_: A course is full when the number of participants reached the maximum amount of participants

```graphql
type Course {
  # bad
  participants: Int!
  maxParticipants: Int!

  # good
  isFull: Boolean!
}
```

### Leveraging the type system

GraphQL makes use of a strict type system in the Schema Definition Languages (SDL) as well as in the query languages (GraphQL). We want to make use of this property instead of creating documentation, style guides or naming conventions.

### Strict conventions

The API should be consistent in the way it is designed and shaped. This style guide will cover most rules to make the API consistent. Sometimes we have to go even further to deliver consistency on a higher level. The team is responsible to make decisions on when to favour consistency over other guidelines.

## Design best practices

### Object types

#### Unique id field

Every output object type that represents an entity should have a single identifier field that uniquely identifies the resource within the type. The identifier field is named `id` and of type `ID!`. The identifier field is owned by the API and while its value should not change for the same resource, the possibility of change should be assumed. The `ID` scalar is serialised as string in GraphQL responses and has to be handled as as such in the frontend code.

### Nullable types

#### Non nullable by default

Generally types should only be nullable if this is strictly necessary. Unnecessarily nullable types create extra null checks in the frontend code and therefore should be avoided.

#### Lists and nullable types

List usually are not nullable. Instead an empty array should represent an empty list. Lists should not contain null values. Instead null values should be filtered out at the API level unless explicitly necessary.

#### Nullable strings

Strings usually are not nullable. Instead the empty string should represent missing information where possible. In some rare cases null should represent the absence of a value while the empty string represents a valid present value.

## Naming

### Types

#### Casing

Type names are in pascal case. That means they start with a capital letter. Type names should not contain whitespace dashes or underscores.

| Examples    |                                                      |
| :---------- | :--------------------------------------------------- |
| Good        | `User`, `SocialSecurityNumber`, `UpdateUserResult`   |
| Not allowed | `user`, `social_security_number`, `updateUserResult` |

#### Singular / Plural

Type names should be singular.

| Examples    |                              |
| :---------- | :--------------------------- |
| Good        | `User`, `UpdateUserResult`   |
| Not allowed | `Users`, `UpdateUserResults` |

#### Output types

Output types should be named according to the style rules and their name in the system. That means that type names should be consistent within the stack and with its usage in the organisation.

Types should not have their kind name (e.g. enum type or scalar type) in their name. If a type is an object type, enum type or scalar is already defined by the type system and does not have to be repeated in the name itself.

| Examples    |                                      |
| :---------- | :----------------------------------- |
| Good        | `Gender`, `CourseStatus`             |
| Not allowed | `GenderEnumType`, `CourseStatusEnum` |

#### Input types

Input object types should be postfixed with `Input`. Input types often behave in different ways than

| Examples    |                                   |
| :---------- | :-------------------------------- |
| Good        | `updateUser` -> `UpdateUserInput` |
| Not allowed | `updateUser` -> `UserUpdate`      |

### Fields

#### Casing

Field names should be in camel case. That means they start with a capital letter. Field names should not contain whitespace, dashes or underscores.

| Examples    |                                                         |
| :---------- | :------------------------------------------------------ |
| Good        | `User`, `SocialSecurityNumber`, `UpdateProfileResult`   |
| Not allowed | `user`, `social_security_number`, `updateProfileResult` |

#### Singular / Plural

Fields that return single (non list type) values should be singular, fields that return list types should be plural.

| Examples    |                                                                  |
| :---------- | :--------------------------------------------------------------- |
| Good        | `users: [User!]!`, `socialSecurityNumber: SocialSecurityNumber!` |
| Not allowed | `user: [User!]!`, `socialSecurityNumbers: SocialSecurityNumber!` |

#### Naming

Fields should be as short as possible and should neither contain the type name of the object type they are defined on nor their output type (sometimes there is a grey zone when the type name is part of the name, e.g. `dateOfBirth`).

| Examples    |                            |
| :---------- | :------------------------- |
| Good        | `name`, `birthday`         |
| Not allowed | `fileName`, `birthdayDate` |

Boolean fields should be prefixed with `is` or `has` (or similar) to create a more speakable field name. This is generally a good practice in most programming languages or database designs and should therefore be aligning well with the other parts of the codebase.

---

### Arguments

#### Casing

Arguments names should be in camel case.

| Examples    |                                  |
| :---------- | :------------------------------- |
| Good        | `id`, `update`, `securityNumber` |
| Not allowed | `ID`, `securityNumber`           |

### Mutations

For mutations all the same rules apply as for normal field types and the following naming conventions on top.

#### Naming

Mutation names should be composed of a verb that expresses the type of action happening and the object name the mutation is operating on.

| Examples    |                                                     |
| :---------- | :-------------------------------------------------- |
| Good        | `updateProfile`, `setCourseStatus`, `trackPageView` |
| Not allowed | `profile`, `setStatus`, `track`                     |

---

### Queries

For queries all the same rules apply as for normal field types and the following naming conventions on top.

### Naming

#### Collection queries

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

#### By id queries

By id queries are queries that query for a single element with a given id. The query name should be the singular of the type name (usually the type name itself since type names must be singular). The return value is nullable and null is returned when a value with the specified id is not found. The field **does not error** (by throwing a exception in the resolver) when the id cannot be located in the database.

_Example_: With the `country` field a single country can be selected by id

```graphql
type Query {
  country(id: ID!): Country
}
```

### Viewer queries

Viewer queries are similar to the collection and by id queries but already have a filter parameter applied: The user id / account id of the viewer. Viewer queries are prefixed with `my` (or `me` for the user or account object query).

_Example_: The `me` query returns the current user object. The `myCourses` query returns all courses that belong to the current viewer.

```graphql
type Query {
  me: Account!
  myCourses: [Course!]!
}
```

## Mutations

Mutations are in a sense very special because coming up with a good mutation design is one of the hardest parts when building a GraphQL schema.

We design three types of mutations to cover all use cases. The types follow strong naming conventions.

### Create Mutations

Create mutations are mutations that create a new entity in the database. The mutation is named after the type of entity it is creating. The mutations must be named in the following schema `create<Entity Name>`. When you want to create a new `User` the mutation to use is the `createUser` mutation.

Create mutations usually take a single required argument `draft` of the draft type of the entity `<Entity Name>Draft`. The draft contains all the arguments that are used to create the instance. This allows us to make all arguments mandatory that are essential for the type and other fields nullable, that contain optional data for the creation.

Examples:

```graphql
type Mutation {
  # good
  createUser(draft: UserDraft!): CreateUserResult!
  createCertificate(draft: CertificateDraft!): CreateCertificateResult!

  # bad
  newUser(user: UserInput!): NewUserResult!
}
```

## Mutation results

Mutation results should contain the name of the mutation and should be postfixed with `Result`.

| Examples    |                                    |
| :---------- | :--------------------------------- |
| Good        | `updateUser` -> `UpdateUserResult` |
| Not allowed | `updateUser` -> `UserUpdate`       |
