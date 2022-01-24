## Tanzu commands



````
tanzu management cluster create --ui
tanzu cluster create tanzutest01 --plan=dev

tanzu standalone-cluster create --ui
tanzu standalone-cluster create tanzu-aws-01 --file /home/frjb/.config/tanzu/tkg/clusterconfigs/tanzu-aws-01.yaml -v 6
tanzu login
tanzu cluster kubeconfig get MY_CLUSTER_NAME_HERE --admin

tanzu package repository add tce-repo --url projects.registry.vmware.com/tce/main:0.9.1 --namespace tanzu-package-repo-global
tanzu package list --all-namespaces
tanzu package available list
tanzu package available list contour.community.tanzu.vmware.com
tanzu package install contour --package-name contour.community.tanzu.vmware.com --version 1.18.1
````
