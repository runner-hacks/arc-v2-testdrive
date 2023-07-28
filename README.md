# arc-v2-testdrive

Code, configuration, and docs to deploy self-hosted runners with [actions-runner-controller v2](https://github.com/actions/actions-runner-controller).


## Resources
- [ARC v2 repo](https://github.com/actions/actions-runner-controller)
- [ARC v2 deep-dive (video)](https://www.youtube.com/watch?v=_F5ocPrv6io&list=PLArH6NjfKsUhvGHrpag7SuPumMzQRhUKY)
- [ARC v2 docs](https://gh.io/arc-docs)
- [controller images](charts/gha-runner-scale-set-controller/values.yaml)


## Deployment Process

### 1. Provision kubernetes cluster

TBA

### 2. Deploy ARC Controller helm chart

```sh
# configure gha-runner-scale-set-controller helm chart values from:
# https://github.com/actions/actions-runner-controller/blob/master/charts/gha-runner-scale-set-controller/values.yaml
TBA

# install gha-runner-scale-set-controller
helm install arc --namespace arc-system \
    --create-namespace -f arc-config/controller/values.yaml \
    --set image.tag="0.4.0" \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller \
    --version "0.4.0" 

# Verify arc-controller pod is Ready and no issues in logs
kubectl get pods -n arc-system
kubectl logs <arc-controller-pod> -n arc-system
```

### 3. Runner ScaleSet preparation

Create GitHub App:

![image](https://github.com/runner-hacks/arc-v2-testdrive/assets/12085451/e0d88fe1-be3e-4599-bea2-b41e05222e23)
![image](https://github.com/runner-hacks/arc-v2-testdrive/assets/12085451/566c1acc-7a02-4236-a6f8-7620300985be)

![image](https://github.com/runner-hacks/arc-v2-testdrive/assets/12085451/368b6e5c-733e-4096-9eef-8f079f36b2ab)
![image](https://github.com/runner-hacks/arc-v2-testdrive/assets/12085451/4f59df6a-a28e-4af8-8313-112e8ab26a07)

> No webhook value is required - ARC v2 does not use webhook-driven scaling

Install into organization for runners:

![image](https://github.com/runner-hacks/arc-v2-testdrive/assets/12085451/ddebe1b5-11d0-4015-8048-887c1399fbb1)

```sh
# create namespace for runners
kubectl create namespace arc-runners

# create k8s secret for github-app
kubectl create secret generic github-app \
    --namespace=arc-runners \
    --from-literal=github_app_id=<app-id> \
    --from-literal=github_app_installation_id=<installation-id> \
    --from-literal=github_app_private_key='<app-private-key-value>'
```

### 4. Deploy Runner ScaleSet

```sh
# configure gha-runner-scale-set helm chart values from:
# https://github.com/actions/actions-runner-controller/blob/master/charts/gha-runner-scale-set/values.yaml

# install gha-runner-scale-set
helm install scale-set-1 --namespace arc-runners \
    -f arc-config/scale-set-1/values.yaml \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set \
    --version 0.4.0
```

### 5. Validate runners

Verify runners available under org, run test workflow, and test scaling
