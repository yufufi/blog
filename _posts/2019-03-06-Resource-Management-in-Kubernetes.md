# Resource Management in Kubernetes

One of the proposed advantages of an orchestration system for microservices like Kubernetes is the _better_ resource allocation.  How does Kubernetes do this?

* Underlying linux mechanism used by Kubernetes for resource management is called `control groups` a.k.a `cgroups`
	* `cgroups` allows limiting, accounting, isolation of resource usage such us CPU, memory, disk I/O, network etc.
* There are two resource types you can manage: cpu & memory:
	* You can `request` certain amount of each resource. This will mainly dictate the behavior of the Kubernetes scheduler for your `pod`. i.e. Your `pod` will only be scheduled to a `node` iff `allocatable` resource is higher than `request`.
	* You can `limit` usage of each resource. This will dictate how operating system deals with your container at runtime.
	* You can set defaults or acceptable `min` or `max` values using `LimitRange` object.
	* Cpu is a compressible resource. 
		* CPU `request` is guaranteed via `cpu. shares` `cgroup` property.  If your container doesn’t  consume as much CPU as it requested at any point, this can be used by other resources, proportional to what they have requested.
		* CPU `limit` is controlled via `cpu.cfs_period_us` and `cpu.cfs_quota_us`.  This is known as CFS Bandwidth Control. You can read this as: within a given `period` (i.e. some microseconds), you’re allowed to consume up to `quota` microseconds. If you consume your share in that period, you get throttled and wait until the next period. 
		* Kubernetes divides CPU by 1000, cgroup by 1024. You might see slightly different values if you connect the pod and look at the values.
	* Memory is a non-compressible resource.
		* Memory `request` is only used by Kubernetes scheduler and no control is done via `cgroups`.
		* Memory `limit` is controlled via `memory.limit_in_bytes`. 
		* If a node is running out of memory, kernel will start killing processes. If a process is using more memory than its set limit, it goes towards the top of the list of candidates.
		* `oom_killer` is responsible of terminating resources when system experience out of memory.
		* `oom_score_adj` helps `oom_killer` decide and this is set by `kubelet` based on `QoS` classes.
* Kubernetes puts pods into different QoS classes based on their resource allocation demands.
	* This is currently calculated based on values set in `request` and `limit`. However, technically QoS system and “Requests and Limits” system are orthogonal to each other. i.e. this can be changed in the future. 
	* If the `request` and `limit` is the same for a `pod` it will end up in the Guaranteed level.
	* If `limit` is not equal to `request` it will end up in the Burstable level.
	* If none are defined it will end up in the Best-Effort level.
	* Main influence of QoS classes seem to be on OOM Score. In short higher levels are less likely to be killed.
* Kubernetes scheduler tracks `Allocatable` property for each node. This is basically whatever is available for your `pods` to be scheduled.
	* `Allocatable` + `eviction-threshold` + `system-reserved` + `Kube-reserved` is equal to what’s available in a node.
	* These can be controlled via parameters passed into `kubelet`.
	* For AKS, following are observed on `kubelet`:
		* `--kube-reserved`: This will reserve some resources for Kubernetes services such as `kubelet`, `container runtime` etc. For one instance this was observed as `cpu=60m,memory=896Mi` 
		* `--enforce-node-allocatable`: This controls if and on what `kubelet` should enforce allocatable values (by eviction etc).  For AKS this is set to `pods`.
		* `--eviction-hard`: This will create some safe limits so that `kubelet` will start evicting `pods` before all resources are drained. In a random AKS node this was set to `memory.available<100Mi,nodefs.available<10%,nodefs.inodesFree<5%`.
		* AKS does not seem to pass in `--system-reserved`, the only hope for system to survive seems to be the `eviction-threshold`.
	* With aks-engine you can configure some of the `kubelet` options related to resource allocation.

## References:
[Reserve Compute Resources for System Daemons - Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/)
[Configure Out Of Resource Handling - Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/out-of-resource/#eviction-policy)
[aks-engine/clusterdefinitions.md at master · Azure/aks-engine · GitHub](https://github.com/Azure/aks-engine/blob/master/docs/topics/clusterdefinitions.md)
[Index of /doc/Documentation/cgroup-v1/](https://www.kernel.org/doc/Documentation/cgroup-v1/)
[Treat your pods according to their needs - three QoS classes in Kubernetes  cloudowski.com](https://cloudowski.com/articles/three-qos-classes-in-kubernetes/)
[Restricting process CPU usage using nice, cpulimit, and cgroups | Scout APM Blog](https://scoutapp.com/blog/restricting-process-cpu-usage-using-nice-cpulimit-and-cgroups)
[community/resource-qos.md at master · kubernetes/community · GitHub](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/resource-qos.md#oom-score-configuration-at-the-nodes)
[Understanding resource limits in kubernetes: memory](https://medium.com/@betz.mark/understanding-resource-limits-in-kubernetes-memory-6b41e9a955f9)
[Understanding resource limits in kubernetes: cpu time](https://medium.com/@betz.mark/understanding-resource-limits-in-kubernetes-cpu-time-9eff74d3161b)
[QoS, “Node allocatable” and the Kubernetes Scheduler · Do not go gentle into this good night. Rage.](https://www.mgasch.com/post/sched-reconcile/)
