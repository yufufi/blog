## Notes:
* In Linux first process is always `pid` `0`  i.e. kernel
* First user space process is always `pid` `1`  i.e. init
* First process in the a new `pid` namespace also gets the `pid` `1` within that namespace
	* From outside (or rather from the parent namespace’s pov it will have a separate `pid`
* The process with `pid` `1` is special:
	* A process whose parent dies automatically gets attached to this process
	* It doesn’t get all the signal handlers hooked up automatically
	* If it dies, everything else in that namespace will be destroyed
	* (And if it dies in the root namespace, kernel will panic and you reboot your machine)
* Docker runs _your_ process (i.e. service etc whatever you’re running in your container as `pid` `1`)
	* Is your process ready to handle aforementioned special attributes?
		* Will it reap the children?
		* Will it terminate them gracefully if it receives a `SIGTERM`?
	* If you pass `--init` to docker it would create an init process for you. (See https://docs.docker.com/engine/reference/run/#specify-an-init-process)
* Rkt automatically runs an init process for (i.e. `systemd`) and your container process runs as `pid` `2`
* By default `pid` namespace is not shared across containers of the same `pod` in `kubernetes`
	* In `v1.13 ` there is a feature turn on process namespace sharing (see https://kubernetes.io/docs/tasks/configure-pod-container/share-process-namespace/). In thise none of you containers will have a process with `pid` `1` but rather it will be in the pause container. (And yup pause container does what a proper `pid` `1` process needs to do.)
	
## References:
For an explanation of pid namespaces see [here](https://hackernoon.com/the-curious-case-of-pid-namespaces-1ce86b6bc900)
For an explanation of pause container see [here](https://www.ianlewis.org/en/almighty-pause-container)
For an explanation of the zombie reaping problem see [here](https://medium.com/@nagarwal/an-init-system-inside-the-docker-container-3821ee233f4b)
[Tini - A tiny but valid `init` for containers](https://github.com/krallin/tini)
[Dumb-init](https://github.com/Yelp/dumb-init)
See [here](https://engineeringblog.yelp.com/2016/01/dumb-init-an-init-for-docker.html) for a blog post by Yelp engineering explaining the reasoning behind `dumb-init` and how it works. 
