---
layout: guide
doc_stub: false
search: true
section: Testing
title: Overview
desc: Testing a GraphQL system
index: 0
redirect_from:
  - /schema/testing
---


So, you've spiked a GraphQL API, and now you're ready to tighten things up and add some proper tests. These guides will help you think about how to ensure stability and compatibility for your GraphQL system.

- [Structure testing]({{ "/testing/schema_structure" | relative_url }}) verifies that schema changes are backwards-compatible. This way, you don't break existing clients.
- [Runtime testing]({{ "/testing/integration_tests" | relative_url }}) exercises the various behaviors of the GraphQL system, making sure that it returns the right data to the right clients.
