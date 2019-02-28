Relationship between multiple containers is a decent way to understand what a `pod` in the world of `k8s` is. As you probably know containers provide [_operating-system-level virtualization_](https://en.wikipedia.org/wiki/Container_(virtualization)). In Linux this virtualization is achieved via [namespaces](https://en.wikipedia.org/wiki/Linux_namespaces). By default, all containers within a `pod` share `network`, and `ipc` namespaces. In `v1.13` you can also set the `shareProcessNamespace` to share the `process` namespace between containers in the same pod. See [here](https://kubernetes.io/docs/tasks/configure-pod-container/share-process-namespace/) for more info on this. 

There is a good article [here](https://www.mirantis.com/blog/multi-container-pods-and-container-communication-in-kubernetes/) with some examples for communication between containers of the same pod leveraging the shared namespaces (and also volumes).

Also see [this](https://github.com/moby/moby/issues/8781) proposal by @brandandburns to make a `pod` a first class object in the docker API. 

