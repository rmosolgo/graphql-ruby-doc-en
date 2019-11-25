This project holds only the _docs_ from GraphQL-Ruby.

You could rename it and use it for translating the docs.

It will be published at `https://{username}.github.io/{repository-name}`, for example: https://rmosolgo.github.io/graphql-ruby-doc-en/

## Background

I started the project by copying the `guides/` directory from graphql-ruby:

```sh
mkdir graphql-ruby-doc-en
cp -r graphql-ruby/guides/ graphql-ruby-doc-en/
```

Then I used my editor's find-replace to remove the custom plugins (`api_doc`, `internal_link`, `open_an_issue`) and replace them with plain markdown. Then I removed `_plugins/`.

## Development

Preview the site with

```sh
bundle install
bundle exec jekyll serve
```

## Publish

Publish this website to GitHub pages by pushing the master branch.

## TODO:

- `1.9.15` is hardcoded as the latest version of GraphQL-Ruby; it would be nice to auto-update that somehow.
- Make a workflow that will watch `graphql-ruby` for changes and somehow notify you when the `/guides` directory is modified -- that way, you know when to update your copy of the docs.
- I assume search won't work; I don't really know how to go about supporting that.
