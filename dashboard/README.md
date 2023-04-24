# [Kubernetes Dashboard](https://github.com/kubernetes/dashboard)
Kubernetes Setup

```sh
# Deploy kubernetes dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Create admin-user serviceaccount
kubectl create sa admin-user -n kubernetes-dashboard

# Bind cluster-admin to admin-user serviceaccount
kubectl create clusterrolebinding admin-user --clusterrole cluster-admin --serviceaccount kubernetes-dashboard:admin-user

# Create kube home directory
mkdir -p ~/.kube

# Create admin-user token
kubectl create token admin-user -n kubernetes-dashboard > ~/.kube/token

# Proxy api server
kubectl proxy

# Copy and paste token from ~/.kube/token
```

[Proxy Dashboard](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login)