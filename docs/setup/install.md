# Installing Rudr

## Prerequisites 

You will need both `kubectl` and `Helm 3` to install Rudr. 

1. Install `kubectl`. The below is for MacOS. For other OS, please go to https://kubernetes.io/docs/tasks/tools/install-kubectl/. 

    ```bash
    curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl"
    ```

2. Install `Helm 3`. The below is copied directly from the [Helm installation guide](https://helm.sh/docs/using_helm/#installing-helm). 

    1. Download your desired version of [Helm 3 from the releases page](https://github.com/helm/helm/releases)
    2. Unpack it (`tar -zxvf helm-v3.0.0-beta.3-darwin-amd64.tar.gz`)
    3. Find the helm binary in the unpacked directory, and move it to its desired destination (`mv macos-amd64/helm /usr/local/bin/helm`)
    4. From there, you should be able to run the client: helm help.

3. As of this writing, the supported versions of Kubernetes are 1.15 and 1.16, so make sure you have a Kubernetes cluster with a compatible version.

## Installing Rudr Using Helm 3

> Note: In its current version, Rudr will only listen for events in one namespace. This will change in the future. For now, though, you must install Rudr into the namespace into which you will deploy Rudr apps. You may install Rudr multiple times on the same cluster as long as you deploy to a different namespace each time.
 
> Tip: As there are some breaking changes (such as Configuration => ApplicationConfiguration, Component => ComponentSchematic), if you reinstall Rudr, make sure your old CRDs are deleted. Helm will not automatically delete CRDs. You must do this with `kubectl delete crd`.
> Note: In its current version, Rudr will only listen for events in one namespace. This will change in the near future.
 
> Tip: As there are some breaking changes, if you reinstall Rudr, make sure your old CRDs are deleted.

1. Helm install Rudr

```console
$ helm install rudr ./charts/rudr --wait --set image.tag=v1.0.0-alpha.1
NAME: rudr
LAST DEPLOYED: 2019-08-08 09:00:07.754179 -0600 MDT m=+0.710068733
NAMESPACE: default
STATUS: deployed

NOTES:
Rudr is a Kubernetes controller to manage Configuration CRDs.

It has been successfully installed.
```

This will install the CRDs and the controller into your Kubernetes cluster.

> Use the `--set image.tag=VERSION` to specify the version that you want installed. If you do not specify a version, the latest unstable developer release will be installed.

### Verifying the Install

You can verify that Rudr is installed by fetching the CRDs:

```console
$ kubectl get crds -l app.kubernetes.io/part-of=core.oam.dev
NAME                                      CREATED AT
applicationconfigurations.core.oam.dev   2019-10-02T19:57:32Z
componentinstances.core.oam.dev          2019-10-02T19:57:32Z
componentschematics.core.oam.dev         2019-10-02T19:57:32Z
scopes.core.oam.dev                      2019-10-02T19:57:32Z
traits.core.oam.dev                      2019-10-02T19:57:32Z
```

You should see at least those five CRDs. You can also verify that the Rudr deployment is running:

```console
$ kubectl get deployment rudr
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
rudr   1/1     1            1           2m47s
```

### Upgrading

To upgrade Rudr, typically you only need to use Helm.

> Tip: During the Alpha and Beta phase of Rudr, we recommend also deleting your CRDs manually. You must do this with `kubectl delete crd`.

```console
$ helm upgrade rudr charts/rudr
```

The above will update your Rudr to the latest version.

### Uninstalling

```console
$ helm delete rudr
```

This will leave the CRDs and configurations intact.

**NOTE: When you delete the CRDs, it will delete everything touching Open Application Model from configurations to secrets.**

```console
kubectl delete crd -l app.kubernetes.io/part-of=core.oam.dev
```

The above will delete the CRDs and clean up everything related with Open Application Model.

## Installing Implementations for Traits

Rudr provides several traits, including ingress and autoscaler. However, it does not install default implementations of some of these. This is because they map to primitive Kubernetes features that can be fulfilled by  different controllers.

The best place to find implementations for your traits is [Helm Hub](https://hub.helm.sh/).

### Manual Scaler

The manual scaler trait has no external dependencies.

### Ingress

To successfully use an `ingress` trait, you will need to install one of the Kubernetes Ingress controllers. We recommend [nginx-ingress](https://hub.helm.sh/charts/stable/nginx-ingress).

```console
$ helm install nginx-ingress stable/nginx-ingress
```

*Note:* You still must manage your DNS configuration as well. Mapping an ingress to `example.com` will not work if you do not also control the domain mapping for `example.com`.

### Autoscaler

To use the autoscaler trait, you must install a controller for Kubernetes `HorizontalPodAutoscaler`s. We recommend [KEDA](https://hub.helm.sh/charts/kedacore/keda-edge).

```
$ helm install keda stable/keda
```

## Running for Development

Developers may prefer to run a local copy of the Rudr daemon. To do so:

1. Make sure the CRDs are installed on your target cluster
2. Make sure your current Kubernetes context is set to your target cluster. Rudr will inherit the credentials from this context entry.
3. From the base directory of the code, run `make run`. This will start Rudr in the foreground, running locally, but listening on the remote cluster.

## Appendix

You could check the [appendix doc](appendix.md) to find more information.
