---
layout: guide
doc_stub: false
search: true
section: Schema
title: Definition
desc: Defining your schema
index: 1
---

A GraphQL system is called a _schema_. The schema contains all the types and fields in the system. The schema executes queries and publishes an [introspection system]({{ "/schema/introspection" | relative_url }}).

Your GraphQL schema is a class that extends `GraphQL::Schema`, for example:

```ruby
class MyAppSchema < GraphQL::Schema
  max_complexity 400
  query Types::Query
  use GraphQL::Batch

  # Define hooks as class methods:
  def self.resolve_type(type, obj, ctx)
    # ...
  end

  def self.object_from_id(node_id, ctx)
    # ...
  end

  def self.id_from_object(object, type, ctx)
    # ...
  end
end
```

There are lots of schema configuration options:

- [root objects, introspection and orphan types](#root-objects-introspection-and-orphan-types)
- [object identification hooks](#object-identification-hooks)
- [execution configuration](#execution-configuration)
- [context class](#context-class)
- [default limits](#default-limits)
- [plugins](#plugins)

For defining GraphQL types, see the guides for those types: [object types]({{ "/type_definitions/objects), [interface types](/type_definitions/interfaces), [union types](/type_definitions/unions),  [input object types](/type_definitions/input_objects), [enum types](/type_definitions/enums), and  [scalar types](/type_definitions/scalars" | relative_url }}).

## Root Objects, Introspection and Orphan Types

A GraphQL schema is a web of interconnected types, and it has a few starting points for discovering the elements of that web:

__Root types__ (`query`, `mutation`, and `subscription`) are the [entry points for queries to the system](https://graphql.org/learn/schema/#the-query-and-mutation-types). Each one is an object type which can be connected to the schema by a method with the same name:

```ruby
class MySchema < GraphQL::Schema
  # Required:
  query Types::Query
  # Optional:
  mutation Types::Mutation
  subscription Types::Subscription
end
```

__Introspection__ is a built-in part of the schema. Every schema has a default introspection system, but you can [customize it]({{ "/schema/introspection" | relative_url }}) and hook it up with `introspection`:

```ruby
class MySchema < GraphQL::Schema
  introspection CustomIntrospection
end
```

__Orphan Types__ are types which should be in the schema, but can't be discovered by traversing the types and fields from `query`, `mutation` or `subscription`. This has one very specific use case, see [Orphan Types]({{ "/type_definitions/interfaces#orphan-types" | relative_url }}).

```ruby
class MySchema < GraphQL::Schema
  orphan_types [Types::Comment, ...]
end
```

## Object Identification Hooks

A GraphQL schema needs a handful of hooks for finding and disambiguating objects while queries are executed.

__`resolve_type`__ is used when a specific object's corresponding GraphQL type must be determined. This happens for fields that return [interface]({{ "/type_definitions/interfaces) or  [union](/type_definitions/unions" | relative_url }}) types. The class method `def self.resolve_type` is used:

```ruby
class MySchema < GraphQL::Schema
  def self.resolve_type(abstract_type, object, context)
    # Disambiguate `object`, from among `abstract_type`'s members
    # (`abstract_type` is an interface or union type.)
  end
end
```

__`object_from_id`__ is used by Relay's `node(id: ID!): Node` field. It receives a unique ID and must return the object for that ID, or `nil` if the object isn't found (or if it should be hidden from the current user).

```ruby
class MySchema < GraphQL::Schema
  def self.object_from_id(unique_id, context)
    # Find and return the object for `unique_id`
    # or `nil`
  end
end
```

__`id_from_object`__ is used to implement Relay's `Node.id` field. It should return a unique ID for the given object. This ID will later be sent to `object_from_id` to refetch the object.

```ruby
class MySchema < GraphQL::Schema
  def self.id_from_object(object, type, context)
    # Return a unique ID for `object`, whose GraphQL type is `type`
  end
end
```

## Execution Configuration

__`instrument`__ attaches instrumenters to the schema, see [Instrumentation]({{ "/queries/instrumentation" | relative_url }}) for more information.

```ruby
class MySchema < GraphQL::Schema
  instrument :field, ResolveTimerInstrumentation
end
```

__`tracer`__ is another way to hook into execution, see [Tracing]({{ "/queries/tracing" | relative_url }}) for more.

```ruby
class MySchema < GraphQL::Schema
  tracer MetricTracer
end
```

__`query_analyzer`__ and __`multiplex_analyzer`__ accept processors for ahead-of-type query analysis, see [Analysis]({{ "/queries/analysis" | relative_url }}) for more.

```ruby
class MySchema < GraphQL::Schema
  query_analyzer MyQueryAnalyzer.new
end
```

__`lazy_resolve`__ registers classes with [lazy execution]({{ "/schema/lazy_execution" | relative_url }}):

```ruby
class MySchema < GraphQL::Schema
  lazy_resolve Promise, :sync
end
```

__`type_error`__ handles type errors at runtime, read more in the [Invariants guide]({{ "/errors/type_errors" | relative_url }}).

```ruby
class MySchema < GraphQL::Schema
  def self.type_error(type_err, context)
    # Handle `type_err` in some way
  end
end
```

__`rescue_from`__ accepts error handlers for application errors, for example:

```ruby
class MySchema < GraphQL::Schema
  rescue_from(ActiveRecord::RecordNotFound) { "Not found" }
end
```

## Context Class

Usually, `context` is an instance of `GraphQL::Query::Context`, but you can create a custom subclass and attach it with `.context_class`, for example:

```ruby
class CustomContext < GraphQL::Query::Context
  # Shorthand to get the current user
  def viewer
    self[:viewer]
  end
end

class MySchema < GraphQL::Schema
  context_class CustomContext
end
```

Then, during execution, `context` will be an instance `CustomContext`.

## Default Limits

`max_depth` and `max_complexity` apply some limits to incoming queries. See [Complexity and Depth]({{ "/queries/complexity_and_depth" | relative_url }}) for more.

`default_max_page_size` applies limits to `Connection` fields.

```ruby
class MySchema < GraphQL::Schema
  max_depth 10
  max_complexity 300
  default_max_page_size 20
end
```

## Plugins

A plugin is an object that responds to `#use`. Plugins are used to attach new behavior to a schema without a lot of API overhead. For example, the gem's [monitoring tools]({{ "/queries/tracing#monitoring" | relative_url }}) are plugins:

```ruby
class MySchema < GraphQL::Schema
  use(GraphQL::Tracing::NewRelicTracing)
end
```
