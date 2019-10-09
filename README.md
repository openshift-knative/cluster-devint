# Logging in to DevInt Cluster

Make sure your GitHub user is a member of the
openshift-knative/cluster-devint team at
https://github.com/orgs/openshift-knative/teams/cluster-devint. Ping
another member of that team to be added if you're not part of
it.

Go to
https://console-openshift-console.apps.devint.openshiftknativedemo.org
and choose the `github` identity provider. Do the login dance with
your GitHub account.

# Using the DevInt Cluster

Create a new project after logging in and deploy what you need. This
is a shared cluster and does not have unlimited resources, so play
nicely.

The first time you deploy a Knative Service in a new namespace it may
take up to a minute for it to become externally accessible. This delay
should go away in future releases as we improve the Service Mesh
integration with our Serverless Operator.

# Suggesting changes to the DevInt Cluster

The entire cluster is managed via [Argo
CD](https://github.com/argoproj/argo-cd) from this repository. If
you'd like a new operator installed on the cluster or anything else
that needs cluster administrator permission to do, file an issue or PR
in this repo with the desired change and someone will merge it after
reviewing.

Once merged the change will be live on the cluster within a few
minutes.

# DevInt Cluster Creation (only relevant for operations)

## Configure AWS CLI to use installer IAM credentials
```
aws configure --profile openshiftknativedemo-installer
AWS Access Key ID [None]: *****
AWS Secret Access Key [None]: *****
Default region name [None]: us-west-2
Default output format [None]:
```

## Create cluster
Using openshift-install 4.1.16, create the install config:

```
mkdir -p ~/Documents/openshift_clusters
cd ~/Documents/openshift_clusters

export AWS_PROFILE=openshiftknativedemo-installer

openshift-install create install-config --dir devint-openshiftknativedemo
```

For both control plane and compute nodes, set instance size to m5.xlarge:

```
  platform:
    aws:
      type: m5.xlarge
```

Now actually create the cluster:

```
openshift-install create cluster --dir devint-openshiftknativedemo --log-level info
```

## Populate Secrets
Download bitwarden CLI from https://help.bitwarden.com/article/cli/#download--install

Login to bitwarden the first time you use it:
```
bw login username@company.com password
```

Unlock the session and optionally pull the latest secrets if things have changed
```
export BW_SESSION=$(bw unlock password --raw)
bw sync # only needed if secrets updated server-side
./populate-secrets-from-bitwarden.sh
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
