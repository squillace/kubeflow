# My First Bundle

This is an empty bundle that porter has created to get you started!

# Contents

error for tomorrow:
```
time="2021-01-05T00:01:39Z" level=info msg="Log cluster name into KfDef: cncf" filename="kustomize/kustomize.go:253"
time="2021-01-05T00:01:39Z" level=info msg="Deploying application namespaces" filename="kustomize/kustomize.go:266"
2021/01/05 00:01:39 absolute path error in '/cnab/app/v1.2-deployment/v1.2-deployment/.cache/manifests/manifests-ebeb4c285a0cf49750c4f07015b58cc27ca4c3a1/namespaces/base' : evalsymlink failure on '/cnab/app/v1.2-deployment/v1.2-deployment/.cache/manifests/manifests-ebeb4c285a0cf49750c4f07015b58cc27ca4c3a1/namespaces/base' : lstat /cnab/app/v1.2-deployment/v1.2-deployment: no such file or directory
time="2021-01-05T00:01:39Z" level=error msg="Error evaluating kustomization manifest for namespaces: accumulating resources: accumulating resources from '../../v1.2-deployment/.cache/manifests/manifests-ebeb4c285a0cf49750c4f07015b58cc27ca4c3a1/namespaces/base': open /cnab/app/v1.2-deployment/v1.2-deployment/.cache/manifests/manifests-ebeb4c285a0cf49750c4f07015b58cc27ca4c3a1/namespaces/base: no such file or directory" filename="kustomize/kustomize.go:155"
Error: failed to apply:  (kubeflow.error): Code 500 with message: kfApp Apply failed for kustomize:  (kubeflow.error): Code 500 with message: error evaluating kustomization manifest for namespaces: accumulating resources: accumulating resources from '../../v1.2-deployment/.cache/manifests/manifests-ebeb4c285a0cf49750c4f07015b58cc27ca4c3a1/namespaces/base': open /cnab/app/v1.2-deployment/v1.2-deployment/.cache/manifests/manifests-ebeb4c285a0cf49750c4f07015b58cc27ca4c3a1/namespaces/base: no such file or directory
```
## porter.yaml

This is the porter manifest. See https://porter.sh/author-bundles/ for 
details on every field and how to configure your bundle. This is a required
file.

## helpers.sh

This is a bash script where you can place helper functions that you can call
from your porter.yaml file.

## README.md

This explains the files created by `porter create`. It is not used by porter and
can be deleted.

## Dockerfile.tmpl

This is a template Dockerfile for the bundle's invocation image. You can
customize it to use different base images, install tools and copy configuration
files. Porter will use it as a template and append lines to it for the mixin and to set
the CMD appropriately for the CNAB specification. You can delete this file if you don't
need it.

Add the following line to **porter.yaml** to enable the Dockerfile template:

```yaml
dockerfile: Dockerfile.tmpl
```

By default, the Dockerfile template is disabled and Porter automatically copies
all of the files in the current directory into the bundle's invocation image. When
you use a custom Dockerfile template, you must manually copy files into the bundle
using COPY statements in the Dockerfile template.

## .gitignore

This is a default file that we provide to help remind you which files are
generated by Porter, and shouldn't be committed to source control. You can
delete it if you don't need it.

## .dockerignore

This is a default file that controls which files are copied into the bundle's
invocation image by default. You can delete it if you don't need it.


Making sure Kubeflow releases for Azure / AKS work would with various combinations of Kubernetes and Istio require the following:

For each version branch of github.com/kubeflow/manifests (e.g. v1.2-branch) deploy the corresponding version azure config file from the kfdef folder using the corresponding version kfctl tool.
So for example, if you are on v1.2-branch, you'd download kfctl version v.1.2.0 from https://github.com/kubeflow/kfctl/releases/tag/v1.2.0. Then using this tool you'd deploy Kubeflow with the manifest file kfdef/kfctl_azure.v1.2.0.yaml.
Once everything is deployed successfully we'd need to run an end to end test. However, I don't know of a good end to end test that checks the deployed Kubeflow installation is working (beyond checking that all components are deployed which isn't sufficient). This is something I'll have to dig in deeper and will discuss in the Kubeflow community slack.
Here is how I am testing at the moment:
```
az aks create --resource-group kubeflow --name kubeflow --node-count 3 --generate-ssh-keys --node-vm-size Standard_DS3_v2 --location westus2
az aks get-credentials -g kubeflow -n kubeflow
ISTIO_VERSION=1.6.14
curl -sL "https://github.com/istio/istio/releases/download/$ISTIO_VERSION/istioctl-$ISTIO_VERSION-osx.tar.gz" | tar xz
chmod +x ./istioctl
PATH=$PATH:$PWD
istioctl operator init
kubectl create ns istio-system
kubectl apply -f istio.aks.yaml   # see attached file which I got from the AKS Istio documentation
```

# Once Istio is up and running proceed with this:
```
curl -sL "https://github.com/kubeflow/kfctl/releases/download/v1.2.0/kfctl_v1.2.0-0-gbc038f9_linux.tar.gz" | tar xz
chmod +x ./kfctl
export CONFIG_URI="https://raw.githubusercontent.com/kubeflow/manifests/v1.2-branch/kfdef/kfctl_azure.v1.2.0.yaml"
mkdir v1.2-deployment
cd v1.2-deployment
kfctl apply -V -f ${CONFIG_URI}
```
# Once Kubeflow is fully deployed and running proceed with his:
```
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```
At this point I go to localhost:8080 and log in with a random username (which automatically created that name as a Kubernetes namespace). Then I start a Jupyter notebook server and make sure it comes up.
