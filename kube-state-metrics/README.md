

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update


kubectl create ns metrics


helm upgrade -i kube-state-metrics -n metrics prometheus-community/kube-state-metrics