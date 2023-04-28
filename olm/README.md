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