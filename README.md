# Kubecost Setup for OpenShift

## Overview

This repository is designed to be an easy guide for the most common configurations of Kubecost in OpenShift clusters. It does not replace our [published documentation](https://guide.kubecost.com/) which will have details for many more use cases.

---
## Usage

Update CLUSTER_NAME in [values-openshift.yaml](values-openshift.yaml)

Then install against the local cost-analyzer repo:

```bash
helm upgrade --install kubecost ./cost-analyzer --namespace kubecost --create-namespace -f ./values-openshift.yaml
```

Wait for all pods to be ready.

Create a route to kubecost-cost-analyzer:9090.
Kubecost will be collecting data, please wait 5-15 minutes before the UI to reflect the resources in the local cluster.