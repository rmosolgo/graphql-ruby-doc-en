---
layout: guide
doc_stub: false
search: true
section: Subscriptions
title: Action Cable Implementation
desc: GraphQL subscriptions over ActionCable
index: 4
---

[ActionCable](https://guides.rubyonrails.org/action_cable_overview.html) is a great platform for delivering GraphQL subscriptions on Rails 5+. It handles message passing (via `broadcast`) and transport (via `transmit` over a websocket).

To get started, see examples in the API docs: `GraphQL::Subscriptions::ActionCableSubscriptions`.

See client usage for [Apollo Client]({{ "/javascript_client/apollo_subscriptions) or [Relay Modern](/javascript_client/relay_subscriptions" | relative_url }}).
