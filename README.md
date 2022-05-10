# K8SAPP: GPU Feature Discovery

Nvidia provides an extension to the standard Kubernetes Node Feature
Discovery to add additional annotations/labels. This K8SAPP is designed
to facilitate its installation and maintenance.

## Application Repo Organization

* `README.md` - description, notes, and references
* `CHANGELOG.md` - changes between vendor versions and customizations
* `check` - check for latest update
* `get` - overwrite `nfd.yaml` with latest

## Application Management Procedures

Avoid putting any of this into a separate script since things might
change upon update and every change needs to be reviewed again and *all*
the management procedures could potentially need to be updated.

### Fetching the Code

TODO

### Reviewing Code and Container Images

TODO

### Customizing and Configuring

TODO

### Installing

TODO

### Updating

TODO

### Uninstalling

TODO

### Check for Updates

TODO

## Related References

* https://github.com/rwxrob/k8sapp-nfd
