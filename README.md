# sbt-graphql [![Build Status](https://travis-ci.org/muuki88/sbt-graphql.svg?branch=master)](https://travis-ci.org/muuki88/sbt-graphql) [ ![Download](https://api.bintray.com/packages/sbt/sbt-plugin-releases/sbt-graphql/images/download.svg) ](https://bintray.com/sbt/sbt-plugin-releases/sbt-graphql/_latestVersion) 

> This plugin is an experiment at this moment.
> SBT 1.x only

SBT plugin to generate and validate graphql schemas written with Sangria.

See also: https://github.com/mediative/sangria-codegen

# Goals

This plugin is intended for testing pipelines that ensure that your graphql
schema and queries are intact and match. You should also be able to compare
it with another schema, e.g. the production schema, to avoid breaking changes.

# Features

All features are based on the excellent [Sangria GraphQL library](http://sangria-graphql.org)

* Schema generation - inspired by [mediative/sangria-codegen](https://github.com/mediative/sangria-codegen)
* Schema validation - [sangria schema validation](http://sangria-graphql.org/learn/#schema-validation)
* Schema validation against another schema - [sangria schema comparison](http://sangria-graphql.org/learn/#schema-comparison)
* Schema release note generation
* Query validation against your locally generated schema - [sangria query validation](http://sangria-graphql.org/learn/#query-validation)

# Usage

Add this to your `plugins.sbt` and replace the `<version>` placeholder with the latest release.

```scala
addSbtPlugin("rocks.muki" % "sbt-graphql" % "<version>")
```

In your `build.sbt` enable the plugins and add sangria. I'm using circe as a parser for my json response.

```scala
enablePlugins(GraphQLSchemaPlugin, GraphQLQueryPlugin)

libraryDependencies ++= Seq(
  "org.sangria-graphql" %% "sangria" % "1.3.0",
  "org.sangria-graphql" %% "sangria-circe" % "1.1.0"
)
``` 

## Schema generation

The schema is generated by accessing the application code via a generated main class that renders
your schema. The main class accesses your code via a small code snippet defined in `graphqlSchemaSnippet`.

Example:
My schema is defined in an object called `ProductSchema` in a field named `schema`.
In your `build.sbt` add

```scala
graphqlSchemaSnippet := "example.ProductSchema.schema"
``` 

Now you can generate a schema with

```bash
$ sbt graphqlSchemaGen
```

## Schema comparison

Sangria provides an API for comparing two Schemas. A change can be breaking or not.
The baseline schema is the one generated by `graphqlSchemaGen`. The other schema is
defined via the `graphqlProductionSchema` task.

`sbt-grapqhl` provides a helper object `SchemaLoader` to load schemas from different
places.

```scala
// from a file
graphqlProductionSchema := SchemaLoader.fromFile((resourceManaged in Compile).value / "prod.graphql")

// from a graphql endpoint via introspection
graphqlProductionSchema := SchemaLoader.fromIntrospection(
  "http://prod.your-graphql.net/graphql",
  streams.value.log
)
```

The introspection query doesn't support headers at this moment, but will be added
soon.

## Schema release notes

`sbt-graphql` creates release notes from changes between the `graphqlSchemaGen` and the
`graphqlProductionSchema`. The format is currently markdown.

```bash
$ sbt graphqlReleaseNotes
```

## Query validation

The query validation uses the schema generated with `graphqlSchemaGen` to validate against all
graphql queries defined under `src/main/graphql`. Using separated `graphql` files for queries
is inspired by [apollo codegen](https://github.com/apollographql/apollo-codegen) which generates
typings for various languages.

To validate your graphql files run

```bash
sbt graphqlValidateQueries
```

You can change the source directory for your graphql queries with this line in
your `build.sbt`

```scala
sourceDirectory in (Compile, graphqlValidateQueries) := file("path/to/graphql")
```

# Developing

## Test project

You can try out your changes immediately with the `test-project`:

```bash
$ cd test-project
sbt
```

If you change code in the plugin you need to `reload` the test-project.

## Releasing

Push a tag `vX.Y.Z` a travis will automatically release it.
If you push to the `snapshot` branch a snapshot version (using the git sha)
will be published.

The `git.baseVersion := "x.y.z"` setting configures the base version for
snapshot releases.