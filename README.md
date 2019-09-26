# DevInt Cluster Creation

## Configure AWS CLI to use installer IAM credentials

```
aws configure --profile openshiftknativedemo-installer
AWS Access Key ID [None]: *****
AWS Secret Access Key [None]: *****
Default region name [None]: us-west-2
Default output format [None]:
```

## Create cluster
using openshift-install 4.1.16

```
mkdir -p ~/Documents/openshift_clusters
cd ~/Documents/openshift_clusters

export AWS_PROFILE=openshiftknativedemo-installer

openshift-install create install-config --dir devint-openshiftknativedemo
```

for both controlplane and compute set instance size to m5.xlarge
  platform:
    aws:
      type: m5.xlarge

```
openshift-install create cluster --dir devint-openshiftknativedemo --log-level info
```


## Install Argo CD
```
oc create namespace argocd
oc apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v1.2.2/manifests/install.yaml
```

download the argocd CLI from https://github.com/argoproj/argo-cd/releases/tag/v1.2.2


## Create the cluster-setup application to seed the cluster
```
oc apply -f https://raw.githubusercontent.com/openshift-knative/cluster-devint/c216b1ea884f0ff9dd9e91635d81c5c5ee35c285/seed.yaml
```

### Verify the cluster-setup application synced successfully
```
oc port-forward svc/argocd-server -n argocd 8080:443
argocd login localhost:8080 --insecure --username admin --password $(oc get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2)

# make sure health status is healthy and sync status is synced
argocd app get cluster-setup
```
