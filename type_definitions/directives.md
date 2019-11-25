---
layout: guide
doc_stub: false
search: true
section: Type Definitions
title: Directives
desc: Special instructions for the GraphQL runtime
index: 10
experimental: true
---


Directives are system-defined keywords that modify execution. All GraphQL systems have at least _two_ directives, `@skip` and `@include`. For example:

```ruby
query ProfileView($renderingDetailedProfile: Boolean!){
  viewer {
    handle
    # These fields will be included only if the check passes:
    ... @include(if: $renderingDetailedProfile) {
      location
      homepageUrl
    }
  }
}
```

Here's how the two built-in directives work:

- `@skip(if: ...)` skips the selection if the `if: ...` value is truthy (`GraphQL::Schema::Directive::Skip`)
- `@include(if: ...)` includes the selection if the `if: ...` value is truthy (`GraphQL::Schema::Directive::Include`)

GraphQL-Ruby also supports custom directives for use with the [interpreter runtime](/queries/interpreter).

## Custom Directives

Custom directives extend `GraphQL::Schema::Directive`:

```ruby
# app/graphql/directives/my_directive.rb
class Directives::MyDirective < GraphQL::Schema::Directive
  description "A nice runtime customization"
end
```

Then, they're hooked up to the schema using `directive(...)`:

```ruby
class MySchema < GraphQL::Schema
  # Custom directives are only supported by the Interpreter runtime
  use GraphQL::Execution::Interpreter
  # Attach the custom directive to the schema
  directive(Directives::MyDirective)
end
```

`GraphQL::Schema::Directive::Feature` and `GraphQL::Schema::Directive::Transform` are included in the library as examples.

### Arguments

Like fields, directives may have [arguments](/fields/arguments) :

```ruby
argument :if, Boolean, required: true,
  description: "Skips the selection if this condition is true"
```

### Runtime hooks

Directive classes may implement the following class methods to interact with the runtime:

- `def self.include?(obj, args, ctx)`: If this hook returns `false`, the nodes flagged by this directive will be skipped at runtime.
- `def self.resolve(obj, args, ctx)`: Wraps the resolution of flagged nodes. Resolution is passed as a __block__, so `yield` will continue resolution. The return value of this method will be treated as the return value of the flagged field.

Looking for a runtime hook that isn't listed here? <a href='https://github.com/rmosolgo/graphql-ruby/issues/new?title="New directive hook: @something"&body= "<!-- Describe how the directive would be used and then how you might implement it --> "'>open an issue</a> to start the conversation!

### Directive object lifecycle

Currently, Directive classes are never initialized. A later version of GraphQL may initialize these objects for runtime application.
