This folder contains sources that can be built from the [CMP that I wrote](../plugin%20source/cmp-plugin.yaml). If you want to test it for yourself, you can simply run the commands I defined in the CMP :
```shell
export ARGOCD_APP_NAME=helloworld
export ARGOCD_APP_NAMESPACE=helloworld
helm dependency build
helm template . --name-template ${ARGOCD_APP_NAME} --include-crds -f values.yaml --namespace $ARGOCD_APP_NAMESPACE > all.yaml
rm -r templates/*
mv all.yaml templates/all.yaml
kustomize build -o templates/all.yaml
helm template . --name-template ${ARGOCD_APP_NAME} --include-crds -f values.yaml --namespace $ARGOCD_APP_NAMESPACE
```