[The old way](https://argo-cd.readthedocs.io/en/stable/user-guide/config-management-plugins/#option-1-configure-plugins-via-argo-cd-configmap) to make a CMP in ArgoCD [has been depracated](https://argo-cd.readthedocs.io/en/latest/operator-manual/upgrading/2.4-2.5/#argocd-cm-plugins-cmps-are-deprecated) since argocd version 2.5 and will be removed in version 2.6. Instead, [ArgoCD recommends using a sidecar container to the `argocd-repo-server` pod](https://argo-cd.readthedocs.io/en/stable/user-guide/config-management-plugins/#option-2-configure-plugin-via-sidecar).<br>
The `cmp-plugin.yaml` follows the suggested way of making a CMP : it's a configmap containing the definition of a plugin that will be mounted onto the sidecar container.<br>
The `sidecar-only.yaml` file contains only the lines I added to the `argocd-repo-server` deployment of my argocd install. It uses [a custom container with helm, kubectl and kustomize binaries](https://github.com/guillheu/kustomize-helm-docker).<br>
This plugin only runs server-side. It will not modify your git repository. That's why removing the initial helm sources is not actually a problem.<br>
<br>
This specific CMP will do the following :<br>
 * pull and build any helm dependency chart as defined in your `Chart.yaml` file
 * generate all the manifests using Helm, using ArgoCD [standard build environment variables](https://argo-cd.readthedocs.io/en/stable/user-guide/build-environment/) and save them all into the `all.yaml` file
 * replace all helm template files with the `all.yaml` file
 * run `kustomize build` and replace the previous `all.yaml` with its output
 * run `helm template` a second time, in case the `kustomization.yaml` file contained Helm instructions
<br>
The two Helm runs are important, as is illustrated in the [example](../example/) : the first run of `helm template` will only catch the container name field in the final manifest, but not the helm fields added by kustomize as defined in [`kustomization.yaml`](../example/kustomization.yaml).<br>
The `> /dev/null` at the end of some commands is necessary to negate printing to stdout ; ArgoCD will actually read stdout and look for the kubernetes manifests there so we need to keep stdout clean.