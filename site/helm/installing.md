---
title: Installing Weave Flux using Helm
menu_order: 20
---

```console
m     m mmmmm  mmmmm    m
#  #  #   #    #   "#   #
" #"# #   #    #mmm#"   #
 ## ##"   #    #        "
 #   #  mm#mm  #        #
```

Flux comes in several parts:

- the command-line client **fluxctl**
- the daemon, **flux** which maintains the state of the cluster
- a service, which runs in Weave Cloud

But you don't need the last to use Flux; you can just run the daemon
and connect to it with the command line client. This page describes
how.

# Get started with Flux using Helm

If you are using Helm already, this guide is for you. By the end
you will have Helm installing Flux in the cluster and deploying
any code changes for you.

## Prerequisites

You will need to have Kubernetes set up. For a quick local test,
you can use `minikube` or `kubeadm`. Any other Kubernetes setup
will work as well though.

*Note:* If you are using `minikube`, be sure to start the
cluster with `--bootstrapper=kubeadm` so you are using RBAC.

Download Helm:

- On MacOS:

  ```sh
  brew install kubernetes-helm
  ```

- On Linux:
  - Download the [latest release](https://github.com/kubernetes/helm/releases/latest), unpack the
    tarball and put the binary in your `$PATH`

Now create a service account and a cluster role binding for Tiller:

```sh
kubectl -n kube-system create sa tiller

kubectl create clusterrolebinding tiller-cluster-rule \
    --clusterrole=cluster-admin \
    --serviceaccount=kube-system:tiller
```

Now you install Tiller and add the Weaveworks Helm repository.

```sh
helm init --skip-refresh --upgrade --service-account tiller
helm repo add weaveworks https://weaveworks.github.io/flux
```

## Set up Flux

```sh
helm install --name flux \
--set helmOperator.create=true \
--set git.url=git@github.com:dholbach/flux-helm-test \
--set git.chartsPath=charts \
--namespace flux \
weaveworks/flux
```

Allow some time for all containers to get up and running. If you're
impatient, run the following command and see the pod creation
process.

```sh
watch kubectl get pods --all-namespaces
```

## Giving write access

At startup Flux generates a SSH key and logs the public key. Find
the SSH public key with:

```sh
export FLUX_POD=$(kubectl get pods --namespace flux -l "app=flux,release=flux" -o jsonpath="{.items[0].metadata.name}")
kubectl -n flux logs $FLUX_POD | grep identity.pub
```

```sh
helm list --namespace test
```

In order to sync your cluster state with git you need to copy the
public key and create a deploy key with write access on your GitHub
repository.

Open GitHub, navigate to your fork, go to **Setting > Deploy keys**(`https://github.com/YOURUSER/flux-example/settings/keys/new`),
click on **Add deploy key**, check **Allow write access**, paste
the Flux public key and click **Add key**.

## Committing a small change

In this example we are using a simple example of a webservice. Here
we will make a small change to its configuration to use a different
message. The easiest way is to your fork of `flux-example` and change
the `msg` argument.

Replace  `YOURUSER` in `https://github.com/YOURUSER/flux-example/blob/master/helloworld-deploy.yaml`
with your Github ID), open the URL in your browser, edit the file,
change the argument value and commit the file.

You can check out the Flux logs with:

```sh
kubectl -n default logs deployment/flux -f
```

The default sync frequency is 5 minutes. This can be tweaked easily.
By observing the logs you can see when the change landed in in the
cluster.

## Confirm the change landed

To access our webservice and check out its welcome message, simply
run:

```sh
kubectl port-forward deployment/helloworld 8080:80 &
curl localhost:8080
```

## Conclusion

As you can see, the actual steps to set up Flux, get our app
deployed, give Flux access to it and see modifications land are
very straight-forward and are a quite natural work-flow.

As a next step, you might want to dive deeper into [how to control
Flux](site/using.md).

Clone the Flux repo and edit the example configuration in
[deploy/flux-deployment.yaml](../../deploy/flux-deployment.yaml). Then
create all the resources defined in the
[deploy-helm directory](../../deploy-helm/):

```sh
$EDITOR ./deploy-helm/helm-operator-deployment.yaml
kubectl apply -f ./deploy-helm
```

You need to connect the helm-operator to the same repository, pointing
the helm-operator to the git path containing Charts. This is achieved by
setting the `--git-url` and `--git-branch` arguments to the same values
as for flux and setting the `--git-charts-path` argument in the
[`helm-operator-deployment.yaml`](../../deploy-helm/helm-operator-deployment.yaml)
manifest.

## Helm operator (Helm users only)

The Kubernetes deployment configuration file
[helm-operator-deployment.yaml](../../deploy-helm/helm-operator-deployment.yaml) runs the
helm operator, but you'll need to edit it first, at least to supply your
own configuration repo (the `--git-repo` as for flux and the `--git-charts-path`
argument).

```sh
$EDITOR helm-operator-deployment.yaml
kubectl create -f flux-helm-release-crd.yaml -f helm-operator-deployment.yaml
```

# Next

Find out how to [control flux](../using.md) in the cluster.
