# Type annotations
Type annotations allow you to customise the structs that Clorinde generates for your rows (and parameters, see [the section below](#parameter-structs)). Furthermore, this allows you to share these types between multiple queries.

To create type annotations, declare them using the `--:` syntax. Type annotations only need to declare the nullable columns. Here's how it looks:

```sql
--: Author(age?)

--! authors : Author
SELECT name, age FROM authors;

--! authors_from_country (country?) : Author
SELECT name, age
FROM authors
WHERE authors.nationality = :country;
```

This will define a struct named `Author` containing typed fields for the `name` and `age` columns (with `age` being nullable). The same struct will be used for the `authors` and `authors_from_country` queries.

```admonish note
Type annotations are declared with this token: `--:`
```

## Derive traits
You can specify additional `#[derive]` traits for the generated struct by declaring them after the type annotation.

```sql
--: Author(age?) : Default, serde::Deserialize

--! authors : Author
SELECT name, age FROM authors;
```

## Custom attributes
You can add custom attributes to generated structs using the `--#` syntax. This allows you to add documentation, conditional compilation directives, or any other Rust attributes.

```sql
--: Author(age?) : Default, serde::Deserialize
--# doc = "Represents an author in the system"
--# cfg_attr(feature = "graphql", derive(async_graphql::SimpleObject))

--! authors : Author
SELECT name, age FROM authors;
```

This will generate:

```rust
#[derive(Default, serde::Deserialize, Debug, Clone, PartialEq)]
#[doc = "Represents an author in the system"]
#[cfg_attr(feature = "graphql", derive(async_graphql::SimpleObject))]
pub struct Author {
    pub name: String,
    pub age: Option<i32>,
}
```

```admonish note
Custom attributes are declared with this token: `--#` and must come after the type annotation.
```

## Inline types
You can also define type inline if you don't plan on reusing them across multiple queries:

```sql
--! authors_from_country (country?) : Author()
SELECT id, name, age
FROM authors
WHERE authors.nationality = :country;
```

Notice how inline types **must** have a set of parenthesis describing their nullable columns. This syntax is often more compact for simple cases. It doesn't have any other special meaning otherwise.

## Parameter structs
Clorinde will **automatically** generate a parameter struct **if it has more than one column**. The name of the parameter struct is based on the name of the query. You can still manually generate a parameter struct using a type annotation or an inline type.

In any case, note that you don't *need* a parameter struct, you can always work directly with the query function (see the section [query usage](./../using_queries/using_queries.md#building-the-query-object)).
