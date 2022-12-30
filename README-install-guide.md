## Kubecost Installation

This repo contains settings that are frequently required in OpenShift and other clusters requiring specific container security constraints.


There are two templates for container SecurityContexts:
[values-restricted-podsecurity.yaml](./cost-analyzer/values-restricted-podsecurity.yaml)
[values-unrestricted-podsecurity.yaml](./cost-analyzer/values-unrestricted-podsecurity.yaml)
One must be used or there will be helm failure for nil pointers.

### Single Cluster Deployment:

```bash
helm upgrade --install kubecost \
 --repo https://raw.githubusercontent.com/kubecost/openshift-helm-chart/main/ cost-analyzer \
 --namespace kubecost --create-namespace \
 -f https://raw.githubusercontent.com/kubecost/openshift-helm-chart/main/values-openshift.yaml
```

### Multi Cluster Deployment:

**1. Customize the configuration:**

_Cluster Name:_
All clusters need to have a unique cluster name, update CLUSTER_NAME and any other configurations in [values-custom.yaml](./cost-analyzer/values-custom.yaml), save to your local repo.

 > Note: All changes to the helm install should be done in to the values-custom.yaml, leaving all other value files as is.

_Shared Storage:_
Each cluster will write to a shared object-store (S3/Thanos compatible). See [this repo](https://github.com/kubecost/poc-common-configurations) for example object-store.yaml configurations under the appropriate provider. Any Thanos supported storage will work.

```bash
oc create secret generic kubecost-thanos --from-file=object-store.yaml=[/path/to/file/]object-store.yaml -n kubecost
```

_Cloud Integration:_
Cloud Integration is optional and only used on a primary cluster. If enabled, Kubecost will pull actual billing from the cloud provider to reconcile the short-term OnDemand prices with actual. See [this repo](https://github.com/kubecost/poc-common-configurations) for examples for each cloud provider.

If not using cloud-integration, comment out the secret in values-custom.yaml

 > Note that on-prem custom pricing is supported as well. Contact us for help with this.

**2. Install Kubecost Primary Cluster**

A Kubecost primary cluster can be run in any account or on-prem- it does not need to run in the "master payer account"

edit values-custom.yaml with any settings that differ from the repo defaults.

```bash
helm upgrade --install kubecost \
 --repo https://raw.githubusercontent.com/kubecost/openshift-helm-chart/main/ cost-analyzer \
 --namespace kubecost --create-namespace \
 -f https://raw.githubusercontent.com/kubecost/openshift-helm-chart/main/cost-analyzer/disable-psps.yaml \
 -f https://raw.githubusercontent.com/kubecost/openshift-helm-chart/main/cost-analyzer/values-thanos.yaml \
 -f https://raw.githubusercontent.com/kubecost/openshift-helm-chart/main/cost-analyzer/values-unrestricted-podsecurity.yaml \
 -f https://raw.githubusercontent.com/kubecost/openshift-helm-chart/main/cost-analyzer/kubecost-primary-cluster-settings.yaml \
 -f path-to/values-custom.yaml
```

**3. Install Kubecost `agent-only` Clusters:

 > Note: values-custom.yaml should have a unique cluster name for each cluster

```bash
helm upgrade --install kubecost \
 --repo https://raw.githubusercontent.com/kubecost/openshift-helm-chart/main/ cost-analyzer \
 --namespace kubecost --create-namespace \
 -f https://raw.githubusercontent.com/kubecost/openshift-helm-chart/main/cost-analyzer/disable-psps.yaml \
 -f https://raw.githubusercontent.com/kubecost/openshift-helm-chart/main/cost-analyzer/values-thanos.yaml \
 -f https://raw.githubusercontent.com/kubecost/openshift-helm-chart/main/cost-analyzer/values-unrestricted-podsecurity.yaml \
 -f https://raw.githubusercontent.com/kubecost/openshift-helm-chart/main/cost-analyzer/kubecost-secondary-cluster-settings.yaml \
 -f path-to/values-custom.yaml
```

### Notes

Wait for all pods to be ready.

Create a route to the service `kubecost-cost-analyzer` on port `9090` of the `kubecost` project. You can learn more about how to do it on your Openshift portal in this [LINK](https://docs.openshift.com/container-platform/3.11/dev_guide/routes.html#:~:text=to%20the%20router.-,Creating%20Routes,Applications%20section%20of%20the%20navigation.&text=The%20new%20route%20inherits%20the,using%20the%20%2D%2Dname%20option.)

Kubecost will be collecting data, please wait 5-15 minutes before the UI to reflect the resources in the local cluster if using a single cluster install. Wait 4 hours if using Thanos.

## Support

For advanced setup or if you have any questions, you can contact us on [Slack](https://join.slack.com/t/kubecost/shared_invite/enQtNTA2MjQ1NDUyODE5LWFjYzIzNWE4MDkzMmUyZGU4NjkwMzMyMjIyM2E0NGNmYjExZjBiNjk1YzY5ZDI0ZTNhZDg4NjlkMGRkYzFlZTU) or email at team@kubecost.com

To participate in our free Enterprise onboarding program, contact us at support@kubecost.com to schedule these sessions!
