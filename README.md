# arc-v2-testdrive

Code, configuration, and docs to deploy self-hosted runners with [actions-runner-controller v2](https://github.com/actions/actions-runner-controller).


## Resources
- [ARC v2 repo](https://github.com/actions/actions-runner-controller)
- [ARC v2 deep-dive (video)](https://www.youtube.com/watch?v=_F5ocPrv6io&list=PLArH6NjfKsUhvGHrpag7SuPumMzQRhUKY)
- [ARC v2 docs](https://gh.io/arc-docs)
- [controller images](charts/gha-runner-scale-set-controller/values.yaml)


## Deployment Process

```sh
# configure gha-runner-scale-set-controller helm chart values

# install gha-runner-scale-set-controller
helm install arc --namespace arc-system \
    --create-namespace -f arc-config/controller/values.yaml \
    --set image.tag="0.4.0" \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller \
    --version "0.4.0" 

# Verify arc-controller pod is Ready and no issues in logs
kubectl get pods -n arc-system
kubectl logs <arc-controller-pod> -n arc-system

# configure gha-runner-scale-set helm chart values

# create namespace for runners
kubectl create namespace arc-runners

# create github-app


# create k8s secret for github-app
kubectl create secret generic github-app \
    --namespace=my_namespace \
    --from-literal=github_app_id=<app-id> \
    --from-literal=github_app_installation_id=<installation-id> \
    --from-literal=github_app_private_key='<app-private-key-value>'

# install gha-runner-scale-set
helm install scale-set-1 --namespace arc-runners \
    -f arc-config/scale-set-1/values.yaml \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set \
    --version 0.4.0

# Verify arc-controller pod is Ready and no issues in logs
kubectl get pods -n arc-runners
kubectl logs <arc-runner-pod> -n arc-runners
kubectl get AutoscalingRunnerSet -n arc-runners
kubectl describe AutoscalingRunnerSet <scale-set-name> -n arc-runners

# Verify runners available under org, run test workflow, and test scaling
```
