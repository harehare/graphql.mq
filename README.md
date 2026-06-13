<h1 align="center">graphql.mq</h1>

A GraphQL SDL (Schema Definition Language) parser implemented as an [mq](https://github.com/harehare/mq) module.

## Features

- Parse `type`, `interface`, `input`, `enum`, `union`, `scalar` definitions
- Field definitions with type references (`!`, `[]`, `[]!`)
- Field arguments
- `implements` relationships
- `#` comment stripping

## Installation

```sh
cp graphql.mq ~/.local/mq/config/
```

### HTTP Import

```sh
mq -I raw 'import "github.com/harehare/graphql.mq" | graphql::graphql_parse(.)' schema.graphql
```

## API

| Function | Description |
|---|---|
| `graphql_parse(input)` | Parse SDL and return an array of type definition dicts |
| `graphql_type_names(input)` | Return all defined type names |
| `graphql_find_type(input, name)` | Find a type definition by name |
| `graphql_fields(input, type_name)` | Return fields of a specific type |

## Example

Given `schema.graphql`:

```graphql
type User {
  id: ID!
  name: String!
  email: String
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  author: User!
  tags: [String]
}

type Query {
  user(id: ID!): User
  posts(limit: Int, offset: Int): [Post!]!
}

enum Role {
  ADMIN
  EDITOR
  VIEWER
}

scalar DateTime
```

```sh
# List all type names
mq -I raw 'import "graphql" | graphql::graphql_type_names(.)' schema.graphql
# => ["User", "Post", "Query", "Role", "DateTime"]

# Get fields of User type
mq -I raw 'import "graphql" | graphql::graphql_fields(., "User") | map(fn(f): f["name"];)' schema.graphql
# => ["id", "name", "email", "posts"]

# Find required fields
mq -I raw 'import "graphql" | graphql::graphql_fields(., "User") | filter(fn(f): f["type"]["required"];) | map(fn(f): f["name"];)' schema.graphql
# => ["id", "name", "posts"]

# List enum values
mq -I raw 'import "graphql" | graphql::graphql_find_type(., "Role") | ."values"' schema.graphql
# => ["ADMIN", "EDITOR", "VIEWER"]

# Find all list-type fields across all types
mq -I raw 'import "graphql" | graphql::graphql_parse(.) | filter(fn(t): t["kind"] == "type";) | map(fn(t): {"type": t["name"], "lists": map(filter(t["fields"], fn(f): f["type"]["list"];), fn(f): f["name"];)};)' schema.graphql
```

## License

MIT
