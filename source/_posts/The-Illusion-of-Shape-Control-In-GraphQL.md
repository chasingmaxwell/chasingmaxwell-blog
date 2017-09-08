---
title: The Illusion of Shape Control In GraphQL
tags:
  - GraphQL
  - APIs
  - JavaScript
  - NodeJS
  - React
date: 2017-09-08 17:33:41
---

GraphQL offers clients a pretty incredible feature-set. The ease with which you can access and control included properties in the query is a breath of fresh air for front-end developers who've fought with complicated queries on more rigid interfaces like JSON API. GraphQL makes it so good, in fact, that it fools you into thinking it does things it doesn't do. For instance, it produces the illusion that you can actually control the response shape _from the client_. Unfortunately, I'm here to burst your bubble... but don't worry, there's good news too.

<!-- more -->

## The Bad News

Despite what you may have thought (and unfortunately what many will tell you), you cannot control shape from the client in GraphQL. Sure, you can do some things that come pretty close. You can rename and duplicate properties, for instance. But you are still fundamentally chained to the shape defined on the server.

## The Good News: Introducing GraphQL Leveler!

Or are you? As someone who's been duped by this shape-shifting mirage, I set out to make true client-side shape control possible. I ended up with [GraphQL Leveler](https://github.com/chasingmaxwell/graphql-leveler) — an NPM package for GraphQL servers which gives their clients the shape shifting functionality they thought the had!

## More Reading

I wrote recently about GraphQL Leveler and the misconception of client-side shape control in my post [GraphQL Leveler: Controlling Shape In The Query](https://www.fourkitchens.com/blog/development/graphql-leveler-controlling-shape-query/) on the Four Kitchens blog. If you need more convincing about the illusion of shape control or if you want to learn more about GraphQL Leveler, check it out!
