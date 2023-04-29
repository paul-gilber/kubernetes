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
kubectl get packagemanifest tektoncd-operator -o jsonpath="{.status.catalogSource}"

# Check catalog source namespace
kubectl get packagemanifest tektoncd-operator -o jsonpath="{.status.catalogSourceNamespace}"


# Create Subscription
cat <<EOF > tektoncd-operator.yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: tektoncd-operator
  namespace: operators
spec:
  channel: <channel-you-want-to-subscribe-to>
  name: <name-of-your-operator>
  source: tektoncd-operator
  sourceNamespace: olm
  installPlanApproval: Automatic
EOF

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

OperatorGroup (og)

## How to know when an update is available
Check subscription 
- currentCSV = latest available version to OLM
- installedCSV = version installed on cluster
