# [Kubernetes OLM Operator](https://olm.operatorframework.io/)
Kubernetes Setup

## [OLM Operator Installation](https://olm.operatorframework.io/docs/getting-started/#installing-olm-in-your-cluster)
```sh
# Install OLM operator sdk
brew install operator-sdk

# Check cluster
kubectl config current-context
kubectl cluster-info

# Install OLM operator
operator-sdk olm install
```

## Cheatsheet
```sh
# List operators available to install
kubectl get packagemanifests -A

# Set operator details
operator_name="tektoncd-operator"
operator_package_ns="olm"

# Check supported installation modes (own, single, multi or all namespaces).
kubectl get packagemanifest -n $operator_package_ns $operator_name -o jsonpath="{.status.channels[0].currentCSVDesc.installModes}" | jq '.[] | select (.supported==true)'

# Check operator catalog source
kubectl get packagemanifest -n $operator_package_ns $operator_name -o jsonpath="{.status.catalogSource}" ; echo

# Check catalog source namespace
kubectl get packagemanifest -n $operator_package_ns $operator_name -o jsonpath="{.status.catalogSourceNamespace}"; echo

# Check channels
kubectl get packagemanifest -n $operator_package_ns $operator_name -o jsonpath="{.status.channels[*].name}" ; echo

# Create Namespace and Operator Group (Optional, only for non-global operator installation)
cat <<EOF > $operator_name.yaml
---
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: foo
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: my-group
  namespace: foo
EOF

# Create Subscription
cat <<EOF > $operator_name.yaml
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: <name-of-your-subscription>/<name-of-your-operator>
  namespace: <namespace-you-want-your-operator-installed-in>
spec:
  channel: <channel-you-want-to-subscribe-to>
  name: <name-of-your-operator>
  source: <name-of-catalog-operator-is-part-of>
  sourceNamespace: <namespace-that-has-catalog>
  installPlanApproval: <Automatic/Manual>
EOF

# Install Operator
kubectl apply -f $operator_name.yaml

# Wait for completion 
watch kubectl get pods -n <namespace-you-want-your-operator-installed-in>
```

## CRDs
ClusterServiceVersion (csv)
- version of a running operator
- metadata included: name, description, version, repo link, labels, icon, owned/required CRDs, cluster requiremetns, and install strategies
- similar to packaging tools, rpm, deb, apk

CatalogSource (catsrc)
- store of metadata OLM can query to discover and install operators and dependencies
- source types: grpc with image (most common), grpc with address, internal or configmap

OperatorCondition (condition)
- communication between OLM and managed operators

Subscription (sub)
- intent to install an operator
- describe which channel of operator package to subscribe
- describe whether to perform updates automatically or manually

InstallPlan (ip)
- defines set  of resources to be created to install or upgrade operator

OperatorGroup (og)
- multitenant configuration to OLM installed operators
- membership
  csv-a is a member of op-a in namespace ns-a if:
  * op-a is the only operator group in ns-a
  * csv-a's install mode support op-a's target namespace set
- namespace selection may be via
  * `selector` namespace label selector
  * `targetNamespaces` namespace list
- global operator group
  * has no `spec.selector` and `spec.targetNamespaces`
- check csv annotations for operatorgroup membership

## How to know when an update is available
Check subscription 
- currentCSV = latest available version to OLM
- installedCSV = version installed on cluster
