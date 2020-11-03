---
title: "Kubernetes: Finalizers and Custom Controllers"
date: 2020-10-25
tags:
- Kubernetes
---

# Authors

- Matthew Doherty
- Philipp Kuntz
- Robert Gogolok

# Introduction

In the [last blog post](https://gogolok.github.io/posts/kubernetes-finalizers-in-custom-resources/)
we provided an introduction to [Kubernetes Finalizers](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#finalizers).

In the introduction we covered the basic mechanics of how finalizers allow
controllers to implement asynchronous pre-delete hooks. Simply put they inform
the Kubernetes control plane that an action needs to take place before
Kubernetes garbage collection logic can be performed.

{{< figure src="container.jpg" alt="Shipping Container" title="https://unsplash.com/@frankiefoto" >}}

In this blog post we take finalizers a step further by performing the resulting
actions of a resource deletion with the help of a custom controller.

# Preparation

Once again we rely in this blog post on
[Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/).
Please ensure it's installed and running.

We will use the same custom resource scenario from our previous example.

In the last blog post we manually applied our custom resource definition
providing a collection of API objects to the Kubernetes API. In this blog post
our custom controller provides this `CustomResourceDefinition` to the Kubernetes
API when we run the controller against the configured Kubernetes cluster as
described in the demo section. For verbosity we provide the
`CustomResourceDefinition` below for reference.

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: serviceinstances.example.com
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: example.com
  # list of versions supported by this CustomResourceDefinition
  versions:
    - name: v1alpha1
      # Each version can be enabled/disabled by Served flag.
      served: true
      # One and only one version must be marked as the storage version.
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
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: serviceinstances
    # singular name to be used as an alias on the CLI and for display
    singular: serviceinstance
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: ServiceInstance
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
    - si
```

In this blog post we will be using a [custom controller](https://github.com/gogolok/kubernetes-controller-finalizers-example)
to demonstrate how a controller can perform an action when notified about the
pending deletion of a service instance.

To use, clone the repository

```shell
git clone https://github.com/gogolok/kubernetes-controller-finalizers-example.git
cd kubernetes-controller-finalizers-example
```
and start the controller via:

```shell
make install && make run ENABLE_WEBHOOKS=false
```

This will run the custom controller that is associated with our custom
resource definition. The design pattern of a
custom resource and a controller is known as a [Kubernetes
Operator](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/).

Now we continue with the practical demo.

# Demo

In the following steps, we will demonstrate a simple case where the reconcile
loop within the controller handles the insertion and deletion of a finalizer
for our service instance resources.

We begin by creating a file named `customresource0.yaml` with the following
custom resource content:

```yaml
apiVersion: "example.com/v1alpha1"
kind: ServiceInstance
metadata:
  name: my-new-service-instance0
spec:
  service: PostgreSQL
  version: "12"
```

Notice that the finalizer does not exist within the above resource as we have delegated
responsibility to the controller which ensures that the resource includes a
finalizer during the first iteration of its reconcile loop.

Then we need to apply it:

```shell
$ kubectl apply -f customresource0.yaml
serviceinstance.example.com/my-new-service-instance0 created
```

The running controller adds the finalizer to the custom resource under the `metadata.finalizers` field.
The finalizer is named `my-finalizer.example.com`. This can be seen after applying the resource with
the following command:

```shell
kubectl get serviceinstance -o yaml
```

You should now see the following included in the custom resource indicating
that the controller has taken responsibility for the resource by inserting the
finalizer into the resource of the `serviceinstance`.

```yaml
...
    finalizers:
    - my-finalizer.example.com
...
```

This is performed by the following code in our controller:

```go
...
	myFinalizerName := "my-finalizer.example.com"
	if serviceinstance.ObjectMeta.DeletionTimestamp.IsZero() {
		// Inserting finalizer if missing from resource
		if err := r.insertFinalizerIfMissing(ctx, log, serviceinstance, myFinalizerName); err != nil {
			return ctrl.Result{}, err
		}
	} else {
...

func (r *ServiceInstanceReconciler) insertFinalizerIfMissing(ctx context.Context, log logr.Logger, instance *examplev1alpha1.ServiceInstance, finalizerName string) error {
	if !containsString(instance.GetFinalizers(), finalizerName) {
		log.Info("Inserting my finalizer")
		instance.SetFinalizers(append(instance.GetFinalizers(), finalizerName))
		if err := r.Update(context.Background(), instance); err != nil {
			log.Error(err, "Failed to insert my finalizer")
			return err
		}
		log.Info("Inserted my finalizer")
	}
	return nil
}
```

The code checks whether the resource is not being deleted. If this is the case
and the resource is missing our finalizer, we update the resource to include
our finalizer. We then immediately return from the Reconcile method since we
updated the resource causing the loop to be triggered again.

Now let's try to delete the resource using:

```shell
$ kubectl delete -f customresource0.yaml
serviceinstance.example.com "my-new-service-instance0" deleted
```

This time you will notice that the resource is able to delete without kubectl
hanging waiting for manual removal of the finalizer from the resource.

In the custom controller logs you should see the following log output:

```
2020-09-24T00:46:40.727+0200	INFO	controllers.ServiceInstance	Removing external resources {"serviceinstance": "default/my-new-service-instance0"}
2020-09-24T00:46:40.727+0200	INFO	controllers.ServiceInstance	Removed external resources	{"serviceinstance": "default/my-new-service-instance0"}
2020-09-24T00:46:40.727+0200	INFO	controllers.ServiceInstance	Removing my finalizer {"serviceinstance": "default/my-new-service-instance0"}
2020-09-24T00:46:40.737+0200	INFO	controllers.ServiceInstance	Removed my finalizer {"serviceinstance": "default/my-new-service-instance0"}
2020-09-24T00:46:40.738+0200	DEBUG	controller	Successfully Reconciled	{"reconcilerGroup": "example.com, "reconcilerKind": "ServiceInstance", "controller": "serviceinstance", "name": "my-new-service-instance0", "namespace": "default"}
2020-09-24T00:46:40.738+0200	INFO	controllers.ServiceInstance	serviceinstance resource not found. Ignoring since object must be deleted	{"serviceinstance": "default/my-new-service-instance0"}
2020-09-24T00:46:40.738+0200	DEBUG	controller	Successfully Reconciled	{"reconcilerGroup": "example.com", "reconcilerKind": "ServiceInstance", "controller": "serviceinstance", "name": "my-new-service-instance0", "namespace": "default"}
```

The reason for this is the custom controller came to our aid and
provided clean-up logic in the event of a deletion on the resource:

```go
    ...
	myFinalizerName := "my-finalizer.example.com"
	if serviceinstance.ObjectMeta.DeletionTimestamp.IsZero() {
    ...
	} else {
		// The object is being deleted.
		if err := r.removeFinalizerIfExists(ctx, log, serviceinstance, myFinalizerName); err != nil {
			return ctrl.Result{}, err
		}
	}
    ...

// removeFinalizerIfExists removes finalizer and updates resource if specified
// finalizer exists
func (r *ServiceInstanceReconciler) removeFinalizerIfExists(ctx context.Context, log logr.Logger, instance *examplev1alpha1.ServiceInstance, finalizerName string) error {
	if containsString(instance.GetFinalizers(), finalizerName) {
		log.Info("Removing external resources")
		if err := r.deleteExternalResources(instance); err != nil {
			log.Error(err, "Failed to remove external resources")
			return err
		}
		log.Info("Removed external resources")

		log.Info("Removing my finalizer.")
		instance.SetFinalizers(removeString(instance.GetFinalizers(), finalizerName))
		if err := r.Update(ctx, instance); err != nil {
			log.Error(err, "Failed to remove my finalizer")
			return err
		}
		log.Info("Removed my finalizer")
	}
	return nil
}
```

The code determines if the resource is being deleted and removes the finalizer
idempotently from the resource if it exists. This ensures that any clean-up
logic that the controller sees as necessary, such as removal of external
resources, is performed prior to the controller relinquishing responsibility
for the resource.

# Conclusion

In this case the logic is very simple, but we could imagine more complex
scenarios. If the deletion timestamp exists in the resource remove the
finalizer so that Kubernetes garbage collection can take place. This serves as
an illustration for the power that custom controllers can provide. In the
deployment of a more complex application or data service we may have other
operations in mind that can be automated using this approach.

