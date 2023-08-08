---
title: "Built-in min and max functions in Go 1.21"
date: 2023-08-08T07:47:12+02:00
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

For **string arguments** the result for `min` is the first argument with the
smallest (or for `max`, largest) value, compared lexically byte-wise.

{{< figure src="min-max-strings.png" alt="min max shadow" title="" >}}

These new built-ins **will not break your existing code** that already
uses the names min and max. Built-ins aren’t keywords,
so `min` and `max` can be shadowed however you like.

{{< figure src="min-max-shadow.png" alt="min max shadow" title="" >}}
