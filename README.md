# K8SAPP: NVIDIA GPU Feature Discovery

Nvidia provides an extension to the standard Kubernetes Node Feature
Discovery to add additional annotations/labels. This K8SAPP is designed
to facilitate its installation and maintenance.

## Application Repo Organization

* `README.md` - description, notes, and references
* `CHANGELOG.md` - changes between vendor versions and customizations
* `helm` - original helm chart from which derived
* `check` - check for latest update

## Application Management Procedures

Keep in mind that NVGFD requires the node drivers be updated on all
nodes. See NVIDIA's own documentation for how to fulfill this
requirement before proceeding with installation.

Also note that this cannot be tested in Kind for lack of a GPU. Although
the installation will work it will not display the labels from
`nvidia.com/` demonstrating success.

### Fetching the Code

The [`helm`](helm) repo contains the version of nvgfd that we are
using for this build. You can get it by simply downloading the YAML
chart file and extracting it.

```
cd helm
curl -sSLO https://nvidia.github.io/gpu-feature-discovery/stable/gpu-feature-discovery-0.5.0.tgz
tar xvf *.tgz
```

You may want to download the `index.yaml` file for comparison when
checking for the latest version (although we use the GitHub version for
that in the `check` script instead).

```
curl -sSLO https://nvidia.github.io/gpu-feature-discovery/index.yaml
```

### Reviewing Code and Container Images

Look through all the template options and installation docs specifically
for things that match with your Node Feature Discovery installation.

### Customizing and Configuring

Create the initial flattened Helm chart.

```
helm template helm/gpu-feature-discovery --set nfd.deploy=false  > nvgfd.yaml
```

By default the Helm installation is set to install the version of NFD
that GFD wants. This will probably conflict with the NFD you have
installed. Be sure to make sure they sync up and that anything related
to NFD installation is removed and always include the `--set
nfd.deploy=false`. Check that your `nvgfd.yaml` file only contains a
single DaemonSet.

This does not provide a release name, which is usually not needed but
will have to be removed from the `nvgfd.yaml` file.

```yaml
  name: release-name-gpu-feature-discovery  # CHANGE TO FOLLOWING
  name: gpu-feature-discovery
```

Remove any references to the release in the annotations and labels and
the "managed by Helm" stuff:

```yaml
    app.kubernetes.io/managed-by: Helm         # DELETE
    app.kubernetes.io/instance: release-name   # DELETE
```

Move the container image from public into your private registry.

TODO add `skopeo` instructions

Change the container YAML properties to match.

```yaml
      containers:
        - image: nvcr.io/nvidia/gpu-feature-discovery:v0.5.0
```

Add resource constraints.

```yaml
spec:
  template:
    spec:
      containers:
        - resources:
            limits:
                cpu: "100m"
                memory: "128Mi"
            requests:
                cpu: "100m"
                memory: "128Mi"
```

### Installing

Be sure to change into the proper cluster/context (dev, prod, etc.)!

```
k apply -f nvgfd.yaml
```

Note that the `--namespace` is not needed since we are installing into
the pre-existing `node-feature-discovery` namespace. 

After installing, verify stuff has installed properly using your
favorite commands.

```
k config set-context --current --namespace node-feature-discovery
k get node <node> -o yaml  |z yq .metadata.labels
```

Notice that you have all the additional `nvidia.com/*` labels.

```
nvidia.com/cuda.driver.major: "460"
nvidia.com/cuda.driver.minor: "32"
nvidia.com/cuda.driver.rev: "03"
...
```

### Updating

* Download the latest `helm/index.yaml`
* Run `git diff helm/index.yaml` to see what has changed
* Fetch and configure new version
* Summarize changes in CHANGELOG.md
* Uninstall previous (see below)
* Install new (see above)

### Uninstalling

There's just one Daemonset to uninstall. Be careful not to blow away
your `node-feature-discovery` namespace if you want to keep that.

```yaml
kubectl delete ds gpu-feature-discovery
```

### Check for Updates

Check that the latest GitHub release has changed (which is what the
`check` script does).

```bash
curl -sSL https://api.github.com/repos/nvidia/gpu-feature-discovery/releases/latest | jq -r .name
```

## Related References

* https://github.com/rwxrob/k8sapp-nfd
* https://github.com/nvidia/gpu-feature-discovery
