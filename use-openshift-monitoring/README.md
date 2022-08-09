# Using OpenShift metrics

Create a new project for the POC
```sh
oc new-project test00
```

Create the metrics-viewer ServiceAccount, that will be used by our application (Deployment resource)
```sh
oc create sa metrics-viewer -n test00
```

Bind the cluster-monitoring-operator clusterRole to the metrics-viewer ServiceAccount
```sh
oc adm policy add-cluster-role-to-user cluster-monitoring-operator -z metrics-viewer -n test00
```

Create the Deployment resource (I've created here [1] a simple Deployment that already references the ServiceAccount metrics-viewer)
```sh
oc apply -f ./busybox-deployment.yaml -n test00
```

Let's call the metrics endpoint from the newly created pod:
```sh
POD=$(oc get pod -l app=reproducer-03198679 -o jsonpath="{.items[0].metadata.name}")
oc exec -ti $POD -- /bin/bash
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -k -s -H "Authorization: Bearer $TOKEN" https://kube-state-metrics.openshift-monitoring.svc.cluster.local:8443/metrics
curl -k -s -H "Authorization: Bearer $TOKEN" https://prometheus-k8s.openshift-monitoring.svc.cluster.local:9091
```