# OLM install TektonCD Operator

## Prerequisites
- [OLM](../README.md#olm-operator-installation)

## Installation
```sh
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
  source: operatorhubio-catalog
  sourceNamespace: olm
  installPlanApproval: Automatic
EOF

# Install
kubectl apply -f tektoncd-operator.yaml
```