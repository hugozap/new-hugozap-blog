---
title: Using NATS(JetStream) as Gorilla session provider in Go
description: 
date: 2022-01-01
tags:
  - golang
  - nats
  - software
layout: layouts/post.njk
---


For a fun side project I'm using NATS, specifically JetStream, its key value storage engine.

I wanted to handle sessions using NATS with the gorilla Golang package, but didn't find a provider.

Turns out building one for it was not hard thanks to the easy to implement gorilla interface for session management.



Usage:

```golang
import "github.com/hugozap/natsstore"


store, err := natsstore.NewStore(js, "myapp", []byte("secret-key"))
```
You can get it from here  here: https://github.com/hugozap/natsstore

