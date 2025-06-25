# Status Fields

## Cluster and Bundle Display States

Clusters and Bundles have different states in each phase of applying Bundles.

### Bundles

**Ready**: Bundles have been deployed and all resources are ready.

**NotReady**: Bundles have been deployed and some resources are not ready.

**Pending**: Bundles are to be processed by Fleet controller. They might be
waiting for the rollout to be resumed if it was paused (see
[Rollout Strategy](./rollout.md)).

**OutOfSync**: Bundles have been synced from Fleet controller, but downstream
agent hasn't synced the change yet.

**Modified**: Bundles have been deployed and all resources are ready, but there
are some changes that were not made from the Git Repository.

**WaitApplied**: Bundles have been synced from Fleet controller and downstream
cluster, but are waiting to be deployed.

**ErrApplied**: Bundles have been synced from the Fleet controller and the
downstream cluster, but there were some errors when deploying the Bundle.

### Clusters

#### Cluster specific states

**WaitCheckIn**: Waiting for agent to report registration information and
cluster status back.

#### States from Bundles

**Ready**: Bundles in this cluster have been deployed and all resources are
ready.

**NotReady**: There are bundles in this cluster that are in NotReady state.

**Pending**: There are bundles in this cluster that are in Pending state.

**OutOfSync**: There are bundles in this cluster that are in OutOfSync state.

**Modified**: There are bundles in this cluster that are in Modified state.

**WaitApplied**: There are bundles in this cluster that are in WaitApplied
state.

**ErrApplied**: There are bundles in this cluster that are in ErrApplied state.

## GitRepo Conditions

**Ready**: The desired state is the current state.

**Active**: TODO Does that really exist?

**GitPolling**: When the remote git repository is being polled for changes or
initial cloning. Contains an error if it fails.

**Reconciling**: The controller is currently working on reconciling the latest
changes.

**Stalled**: The controller has encountered an error during the reconcile
process or it has made insufficient progress (timeout).

**Accepted**: All GitRepo restrictions could be applied and external helm
secrets exist.

## GitRepo Display States

GitRepo and HelmOp resources share a common base for their statuses. That is the
`StatusBase` struct.

TODO remove Go-specifics and let it look like docs?

```go
type StatusBase struct {
	// ReadyClusters is the lowest number of clusters that are ready over
	// all the bundles of this resource.
	// +optional
	ReadyClusters int `json:"readyClusters"`
	// DesiredReadyClusters	is the number of clusters that should be ready for bundles of this resource.
	// +optional
	DesiredReadyClusters int `json:"desiredReadyClusters"`
	// Summary contains the number of bundle deployments in each state and a list of non-ready resources.
	Summary BundleSummary `json:"summary,omitempty"`
	// Display contains a human readable summary of the status.
	Display StatusDisplay `json:"display,omitempty"`
	// Conditions is a list of Wrangler conditions that describe the state
	// of the resource.
	Conditions []genericcondition.GenericCondition `json:"conditions,omitempty"`
	// Resources contains metadata about the resources of each bundle.
	Resources []Resource `json:"resources,omitempty"`
	// ResourceCounts contains the number of resources in each state over all bundles.
	ResourceCounts ResourceCounts `json:"resourceCounts,omitempty"`
	// PerClusterResourceCounts contains the number of resources in each state over all bundles, per cluster.
	PerClusterResourceCounts map[string]*ResourceCounts `json:"perClusterResourceCounts,omitempty"`
}
```

### ReadyBundleDeployments

`readyBundleDeployments` is a string in the form "%d/%d", that describes the
number of ready bundle deployments over the total number of bundle deployments.

`state` is the state of the GitRepo, e.g. "GitUpdating" or the maximal
BundleState according to StateRank.

`message` contains the relevant message from the deployment conditions.

`error` is true if an error message is present.

## Resources List

The resources lists contain the deployed resources, categorized under `Bundles`
and `GitRepos`.

### Bundles

The deployed resources within bundles can be found in `status.ResourceKey`. This
key represents the actual resources deployed via `bundleDeployments`.

### GitRepos

Similar to bundles, the deployed resources in `GitRepos` are listed in
`status.Resources`. This list is also derived from `bundleDeployments`.

## Resource Counts

This shows how resource counts are propagated from one resource to another:
![Status Propagation](/img/FleetStatusSource.png)

### GitRepos

The `status.ResourceCounts` list for GitRepos is derived from
`bundleDeployments`.

### Clusters

In Clusters, the `status.ResourceCounts` list is derived from GitRepos.

### ClusterGroups

In ClusterGroups, the `status.ResourceCounts` list is also derived from
GitRepos.
