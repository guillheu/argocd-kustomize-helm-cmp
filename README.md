# Kustomize-Helm CMP for ArgoCD
Argocd Config Management Plugin (CMP) that merges Helm and Kustomize deployments.<br>
While ArgoCD allows the use of [both Helm and Kustomize](https://argo-cd.readthedocs.io/en/stable/user-guide/application_sources/) as sources for Kubernetes manifests, using them in conjunction with each other can be troublesome.<br>
In this repo, I will try to explain some of the design choices behind my Kustomize-Helm CMP, and provide the sources for the CMP I'm using in my home lab.


## Why use both Helm and Kustomize ?

This question has been asked and answered before. My reason for wanting to use both is that they serve different purposes.<br>

### Helm
Helm is unavoidable when using ArgoCD because of the amount of core applications ([sealed-secrets](https://github.com/bitnami-labs/sealed-secrets), [cert-manager](https://github.com/cert-manager/cert-manager), [prometheus](https://github.com/bitnami/charts/tree/main/bitnami/kube-prometheus)...) that all have reliable helm charts. It allows for reliable configurable deployment of very complex apps.<br>
However, Helm does not make it easy to modify existing helm charts outside of what's configurable through the values files. Any such change requires you to do a full fork of the chart, then to apply your changes to it. Feasable but cumbersome.

### Kustomize
Kustomize lets you modify anything in existing kubernetes manifests. You could use it to configure a complex app just like you do with Helm's values configuration, but that would take a lot more time and is overall a waste of time for very complex apps. However, it is fantastic at applying a small patch to any kubernetes resource (for instance, adding a sidecar container to an existing deployment, modifying a configmap, changing the container image used in a pod etc...) <u>without touching the original file<u>! This last detail is extremely important when doing GitOps, since it lets you keep the original manifests on your repo.

#### Doesn't Kustomize have an `--enable-helm` option ?
Yes it does, but using it does not fit my workflow that's centered around Helm. It requires rearranging my file structure and <u>systematically<u> using kustomize to build apps, even when there is nothing to kustomize (or have two different kinds of app structures that you'd need to migrate from one to the other).