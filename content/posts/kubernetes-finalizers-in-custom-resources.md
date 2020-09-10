---
title: "Kubernetes: Finalizers in Custom Resources"
date: 2020-09-10
tags:
- Kubernetes
---

# Authors

- Matthew Doherty
- Philipp Kuntz
- Robert Gogolok

# Introduction

When extending the Kubernetes API with [CustomResourceDefinitions](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)
you’ll come  across the dilemma to clean up external resources when deleting a
custom resource. Although you can create a custom resource simply to store and
retrieve structured data, most of the time there is some entity involved,
like custom controllers . The controller will manage this resource and create
other external resources to handle the semantics of that resource. Those
external resources should not live forever once the custom resource does not
exist anymore.

In the following text we’ll work with a custom resource example that represents
a data service instance. That data service instance could be, for example an
instance running a PostgreSQL database. That service instance might store data
to an external blob store, for instance AWS S3 during a backup operation. Once
you want to get rid of this custom resource and therefore that service
instance, you might want to clean up the backups that were created specifically
for this data service instance (for example on AWS S3).

This is where [Kubernetes Finalizers](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#finalizers)
come into play and help to clean up external resources before the deletion of a
custom resource. You can add finalizers to a custom resource which will
prevent for example the kubectl tool from deleting it.

# Demo

Let’s do a practical demo with [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)
on how finalizers can help to prevent the deletion of custom resources.

You can install Minikube using the [Install Minikube instructions](https://kubernetes.io/docs/tasks/tools/install-minikube/).
Once it is up and running, you should be able to call `kubectl get crd` without
an error.

In order to create a custom resource, we first need to create a custom resource definition.
Please copy the following content to a file named `customresourcedefinition.yaml`:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: serviceinstances.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                service:
                  type: string
                version:
                  type: string
  scope: Namespaced
  names:
    plural: serviceinstances
    singular: serviceinstance
    kind: ServiceInstance
    shortNames:
    - si
```

It is a custom resource definition for the above-mentioned example of
PostgreSQL service instances. 

After creating the file, we’re ready to upload our custom resource definition to Kubernetes:

```shell
$ kubectl apply -f customresourcedefinition.yaml
customresourcedefinition.apiextensions.k8s.io/serviceinstances.example.com created
```

Next we’ll create a custom resource fitting our custom resource definition.
Create the following content in a file named `customresource0.yaml`:

```yaml
apiVersion: "example.com/v1"
kind: ServiceInstance
metadata:
  name: my-new-service-instance0
  finalizers:
  - my-finalizer.example.com
spec:
  service: PostgreSQL
  version: "12"
```

Then we apply the custom resource using:

```shell
$ kubectl apply -f customresource0.yaml
serviceinstance.example.com/my-new-service-instance0 created
```

Under `metadata.finalizers` we’ve added an entry for a finalizer called
`my-finalizer.example.com`.

So far this doesn’t play a role and a new ServiceInstance resource has been
created with the name `my-custom-resource0`.

We can get the resource’s yaml representation using:

```shell
$ kubectl get si my-new-service-instance0 -o yaml
...
apiVersion: example.com/v1
kind: ServiceInstance
metadata:
  ...
  creationTimestamp: "2020-09-09T21:36:56Z"
  finalizers:
  - my-finalizer.example.com
  ...
  name: my-new-service-instance0
  ...
spec:
  service: PostgreSQL
  version: "12"
```

Let's try to delete the resource using:

```shell
$ kubectl delete -f customresource0.yaml
serviceinstance.example.com "my-new-service-instance0" deleted
```

After outputting the delete line, kubectl is hanging.

In another shell we can now output the yaml representation of that custom
resource again using:

```shell
$ kubectl get si my-new-service-instance0 -o yaml
apiVersion: example.com/v1
kind: ServiceInstance
metadata:
  ...
  deletionTimestamp: "2020-09-09T21:52:00Z"
  ...
```


Kubernetes has added the field `metadata.deletionTimestamp` to signal the
intention is to delete that resource. The finalizer entry we’ve added is
preventing Kubernetes from deleting that custom resource.

In order to get rid of the resource, we need to remove the finalizer entry and
signal this way we’ve removed external resources locked by that finalizer name.

Let’s edit the file `customresource0.yaml` and remove the finalizer. The file
should now look similar to the following content:

```yaml
apiVersion: "example.com/v1"
kind: ServiceInstance
metadata:
  name: my-new-service-instance0
spec:
  service: PostgreSQL
  version: "12"
```

Let’s apply the changes:

```shell
$ kubectl apply -f customresource0.yaml
serviceinstance.example.com/my-new-service-instance0 configured
```

When we switch back to the hanging `kubectl` command, we can see it succeeded.
The custom resource has been removed since the list of finalizers is empty.

The implications for Kubernetes are that all finalizers have been executed and
have done their job.

# Conclusion

Specifying finalizers can prevent a custom resource from deletion. This gives
the opportunity to clean up external resources associated with the custom
resource.

In a future article, we’ll extend our knowledge to Kubernetes operators and how
to protect custom resources with finalizers during reconciliation.
