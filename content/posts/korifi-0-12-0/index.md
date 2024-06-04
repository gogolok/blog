---
title: "Korifi 0.12.0"
date: 2024-06-04T16:54:03+02:00
tags:
- Go
---

The Korifi community [has released Korifi 0.12.0](https://github.com/cloudfoundry/korifi/releases/tag/v0.12.0).
If you want to quickly try it out, please check the [install Korifi on kind](https://github.com/cloudfoundry/korifi/blob/main/INSTALL.kind.md) guide.

{{< figure src="korifi-logo.png" alt="Korifi logo" title="" >}}

## What is Korifi

[Korifi is an offering built by the Cloud Foundry community](https://www.cloudfoundry.org/technology/korifi/).
Korifi’s purpose is to deliver an inherently higher order abstraction over Kubernetes,
ultimately enabling developers to focus on building applications.

## What's Changed?

For the official relese changes, take a look at the [release notes for Korifi 0.12.0](https://github.com/cloudfoundry/korifi/releases/tag/v0.12.0).

In the following I highlight my contributions.
If you're looking for code and not the short overview here, you can also check [my commit activity to the main branch on GitHub](https://github.com/cloudfoundry/korifi/commits?author=gogolok&since=2024-02-01&until=2024-06-04).

## Add /v3/info Endpoint

Korifi tries to be Cloud Foundry [V3 API compatible](https://github.com/cloudfoundry/korifi/blob/main/docs/api.md).
The **v3/info** endpoint has been implemented in the pull requests [#3239](https://github.com/cloudfoundry/korifi/pull/3239), [#3284](https://github.com/cloudfoundry/korifi/pull/3248), [#3258](https://github.com/cloudfoundry/korifi/pull/3258).

{{< figure src="carbon-info.png" alt="Korifi info" title="" >}}

The Korifi helm chart allows to feed the following properties:

- *minimum* and *recommended* CF CLI version
- *description* and *name* properties
- *custom* labels

The *build* and *version* fields have identical values and are set to the Korifi [version.Version](https://github.com/cloudfoundry/korifi/blob/06142329763e18dbde77fcfad8388a93d28c53dc/version/version.go#L12) variable.

## Allow Pre-Release CF CLI Versions

If you're a developer and live on the cutting edge, you might want to run beta versions of the [CF CLI](https://github.com/cloudfoundry/cli).
In previous versions you would get an error if you would try to use CF CLI beta releases with Korifi.

```
Setting API endpoint to https://localhost...
CF CLI version 9.0.0-beta+334489f01.2024-05-14 is not supported. Korifi requires CF CLI >= 8.5.0
FAILED
```

This has been addressed in the pull request [#3255](https://github.com/cloudfoundry/korifi/pull/3255).

## Add /v3/apps/:guid/processes/:type/stats endpoint

Another endpoint to list the stats for a process type has been added in PR #3307. It provides the same information as the endpoint GET /v3/processes/:guid/stats.

{{< figure src="carbon-stats.png" alt="Korifi stats" title="" >}}
