# OLM install TektonCD Operator

## Prerequisites
- [OLM](../README.md#olm-operator-installation)

## Installation
```sh
# List operators available to install
kubectl get packagemanifests -A

# Set operator details
operator_name="elastic-cloud-eck"
operator_install_ns="elastic"
operator_package_ns="olm"

# Check supported installation modes (own, single, multi or all namespaces).
kubectl get packagemanifest -n $operator_package_ns $operator_name -o jsonpath="{.status.channels[0].currentCSVDesc.installModes}" | jq '.[] | select (.supported==true)'

# Check operator catalog source
kubectl get packagemanifest -n $operator_package_ns $operator_name -o jsonpath="{.status.catalogSource}" ; echo

# Check catalog source namespace
kubectl get packagemanifest -n $operator_package_ns $operator_name -o jsonpath="{.status.catalogSourceNamespace}"; echo

# Check channels
kubectl get packagemanifest -n $operator_package_ns $operator_name -o jsonpath="{.status.channels[*].name}" ; echo


# Check global operators namespace (operators)
kubectl get og -A | grep -i global-operators 

# Create Namespace and Operator Group (Optional, only for non-global operator installation)
cat <<EOF > $operator_name.yaml
---
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: $operator_install_ns
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: $operator_install_ns
  namespace: $operator_install_ns
EOF

# Create Subscription
cat <<EOF >> $operator_name.yaml
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: $operator_name
  namespace: $operator_install_ns
spec:
  channel: stable
  name: $operator_name
  source: operatorhubio-catalog
  sourceNamespace: olm
  installPlanApproval: Automatic
EOF

# Install
kubectl apply -f $operator_name.yaml
```

## Elasticsearch and Kibana Instance
```sh
# https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-openshift-deploy-elasticsearch.html

# https://www.elastic.co/guide/en/cloud-on-k8s/master/k8s-beat-configuration-examples.html#k8s_openshift_monitoring

cat <<EOF > es-kibana.yaml
---
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elasticsearch-sample
  namespace: elastic
spec:
  version: 8.7.0
  nodeSets:
  - name: default
    count: 1
    config:
      node.store.allow_mmap: false
---
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: kibana-sample
  namespace: elastic
spec:
  version: 8.7.0
  count: 1
  elasticsearchRef:
    name: elasticsearch-sample
  podTemplate:
    spec:
      containers:
      - name: kibana
        resources:
          limits:
            memory: 1Gi
            cpu: 1
EOF

# Create metricbeat, filebeat, elasticsearch, kibana
kubectl apply -f es-kibana.yaml
kubectl apply -f filebeat-autodiscover.yaml
kubectl apply -f metricbeat_hosts.yaml

# Get Kibana Instance credentials, username: elastic
kubectl get secret elasticsearch-sample-es-elastic-user -n elastic -o jsonpath='{.data.elastic}' | base64 -d ; echo
```

## References
- [Metric Beat Kubernetes Modules](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-module-kubernetes.html)

