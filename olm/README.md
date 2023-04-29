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

# Install an tektoncd-operator operator for testing
# Check supported installation modes.
# Check whether own, single, multi, or all namespace installation mode is supported.
kubectl get packagemanifest tektoncd-operator -o jsonpath="{.status.channels[0].currentCSVDesc.installModes}" | jq '.[] | select (.supported==true)'

# Check operator catalog source
kubectl get packagemanifest tektoncd-operator -o jsonpath="{.status.catalogSource}" ; echo

# Check catalog source namespace
kubectl get packagemanifest tektoncd-operator -o jsonpath="{.status.catalogSourceNamespace}"; echo

# Check channels
kubectl get packagemanifest tektoncd-operator -o jsonpath="{.status.channels[*].name}" ; echo

# Check global operators namespace (operators)
kubectl get og -A | grep -i global-operators 

# Create Subscription
cat <<EOF > tektoncd-operator.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: tektoncd-operator
  namespace: operators
spec:
  channel: alpha
  name: tektoncd-operator
  source: <name-of-catalog-operator-is-part-of>
  sourceNamespace: <namespace-that-has-catalog>
  installPlanApproval: <Automatic/Manual>
EOF

cat <<EOF > tektoncd-operator.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: tektoncd-operator
  namespace: operators
spec:
  channel: alpha
  name: tektoncd-operator
  source: operatorhubio-catalog
  sourceNamespace: olm
  installPlanApproval: Automatic
EOF

# Install tektoncd-operator
kubectl apply -f tektoncd-operator.yaml

# Wait for completion 
watch kubectl get pods -n operators -l app=tekton-operator

# Create Operator Group (Optional based on installModes)


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
