---
title: "Built-in min and max functions in Go 1.21"
date: 2023-08-01T00:24:12+02:00
draft: true
tags:
- Go
---

Go 1.21 brings the two built-in functions `min` and `max`
to compute the smallest or largest value of a fixed number
of arguments of ordered types.

{{< figure src="min-max-example.png" alt="min max example" title="" >}}

Only a **fixed number of arguments** is supported.
Go needs to know the number of arguments at compile time.

{{< figure src="min-max-error.png" alt="min max error" title="" >}}