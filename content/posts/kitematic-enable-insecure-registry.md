---
title: "Enable Insecure Registry on Kitematic"
date: 2016-02-09T23:47:55+02:00
tags:
- Docker
---

In order to use an insecure registry with [Kitematic](https://kitematic.com/), you can follow the steps below:


* Login to the Kitematic box, e.g. double click in VirtualBox on the kitematic machine that is named 'default' by default :-)
* Edit the file `/var/lib/boot2docker/profile` and append to `EXTRA_ARGS` the `--insecure-registry` parameter:

{{< highlight sh >}}
EXTRA_ARGS='
...
--insecure-registry myregistrydomain.com:5000
'
{{< / highlight >}}

* Restart docker `/usr/local/etc/init.d/docker restart`

You should now be able to push an image to your insecure registry.
