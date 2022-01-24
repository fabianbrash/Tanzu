## Tanzu commands



````
tanzu management cluster create --ui
tanzu cluster create tanzutest01 --plan=dev

tanzu standalone-cluster create --ui
tanzu login
tanzu cluster kubeconfig get MY_CLUSTER_NAME_HERE --admin

tanzu package repository add tce-repo --url projects.registry.vmware.com/tce/main:0.9.1 --namespace tanzu-package-repo-global
tanzu package list --all-namespaces
tanzu package available list
tanzu package available list contour.community.tanzu.vmware.com
tanzu package install contour --package-name contour.community.tanzu.vmware.com --version 1.18.1
````
