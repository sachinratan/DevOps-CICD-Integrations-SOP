### ArgoCD - GitOps CD for Kubernetes, Installation, Setup, Configurations

#### ArgoCD Installation:
- Create namespace for ArgoCD deployment.
  ```
  kubectl create namespace argocd

  namespace/argocd created
  ```
- Deploy ArgoCD
  ```
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
  customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
  customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
  serviceaccount/argocd-application-controller created
  serviceaccount/argocd-applicationset-controller created
  serviceaccount/argocd-dex-server created
  serviceaccount/argocd-notifications-controller created
  serviceaccount/argocd-redis created
  serviceaccount/argocd-repo-server created
  serviceaccount/argocd-server created
  role.rbac.authorization.k8s.io/argocd-application-controller created
  role.rbac.authorization.k8s.io/argocd-applicationset-controller created
  role.rbac.authorization.k8s.io/argocd-dex-server created
  role.rbac.authorization.k8s.io/argocd-notifications-controller created
  role.rbac.authorization.k8s.io/argocd-redis created
  role.rbac.authorization.k8s.io/argocd-server created
  clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
  clusterrole.rbac.authorization.k8s.io/argocd-applicationset-controller created
  clusterrole.rbac.authorization.k8s.io/argocd-server created
  rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
  rolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
  rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
  rolebinding.rbac.authorization.k8s.io/argocd-notifications-controller created
  rolebinding.rbac.authorization.k8s.io/argocd-redis created
  rolebinding.rbac.authorization.k8s.io/argocd-server created
  clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
  clusterrolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
  clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
  configmap/argocd-cm created
  configmap/argocd-cmd-params-cm created
  configmap/argocd-gpg-keys-cm created
  configmap/argocd-notifications-cm created
  configmap/argocd-rbac-cm created
  configmap/argocd-ssh-known-hosts-cm created
  configmap/argocd-tls-certs-cm created
  secret/argocd-notifications-secret created
  secret/argocd-secret created
  service/argocd-applicationset-controller created
  service/argocd-dex-server created
  service/argocd-metrics created
  service/argocd-notifications-controller-metrics created
  service/argocd-redis created
  service/argocd-repo-server created
  service/argocd-server created
  service/argocd-server-metrics created
  deployment.apps/argocd-applicationset-controller created
  deployment.apps/argocd-dex-server created
  deployment.apps/argocd-notifications-controller created
  deployment.apps/argocd-redis created
  deployment.apps/argocd-repo-server created
  deployment.apps/argocd-server created
  statefulset.apps/argocd-application-controller created
  networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
  networkpolicy.networking.k8s.io/argocd-applicationset-controller-network-policy created
  networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
  networkpolicy.networking.k8s.io/argocd-notifications-controller-network-policy created
  networkpolicy.networking.k8s.io/argocd-redis-network-policy created
  networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
  networkpolicy.networking.k8s.io/argocd-server-network-policy created
  ```
- Verify the ArgoCD resources status
  ```
  kubectl get all -n argocd
  NAME                                                   READY   STATUS    RESTARTS       AGE
  pod/argocd-application-controller-0                    1/1     Running   0              2m25s
  pod/argocd-applicationset-controller-fc5545556-58w2q   1/1     Running   0              2m26s
  pod/argocd-dex-server-f59c65cff-pscrs                  1/1     Running   2 (2m8s ago)   2m26s
  pod/argocd-notifications-controller-59f6949d7-ctw8g    1/1     Running   0              2m26s
  pod/argocd-redis-75c946f559-b956d                      1/1     Running   0              2m26s
  pod/argocd-repo-server-6959c47c44-jbqtz                1/1     Running   0              2m26s
  pod/argocd-server-65544f4864-tbrh4                     1/1     Running   0              2m26s
  
  NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
  service/argocd-applicationset-controller          ClusterIP   172.20.10.7      <none>        7000/TCP,8080/TCP            2m27s
  service/argocd-dex-server                         ClusterIP   172.20.67.237    <none>        5556/TCP,5557/TCP,5558/TCP   2m26s
  service/argocd-metrics                            ClusterIP   172.20.73.91     <none>        8082/TCP                     2m26s
  service/argocd-notifications-controller-metrics   ClusterIP   172.20.144.179   <none>        9001/TCP                     2m26s
  service/argocd-redis                              ClusterIP   172.20.116.176   <none>        6379/TCP                     2m26s
  service/argocd-repo-server                        ClusterIP   172.20.106.106   <none>        8081/TCP,8084/TCP            2m26s
  service/argocd-server                             ClusterIP   172.20.180.45    <none>        80/TCP,443/TCP               2m26s
  service/argocd-server-metrics                     ClusterIP   172.20.175.39    <none>        8083/TCP                     2m26s
  
  NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/argocd-applicationset-controller   1/1     1            1           2m26s
  deployment.apps/argocd-dex-server                  1/1     1            1           2m26s
  deployment.apps/argocd-notifications-controller    1/1     1            1           2m26s
  deployment.apps/argocd-redis                       1/1     1            1           2m26s
  deployment.apps/argocd-repo-server                 1/1     1            1           2m26s
  deployment.apps/argocd-server                      1/1     1            1           2m26s
  
  NAME                                                         DESIRED   CURRENT   READY   AGE
  replicaset.apps/argocd-applicationset-controller-fc5545556   1         1         1       2m26s
  replicaset.apps/argocd-dex-server-f59c65cff                  1         1         1       2m26s
  replicaset.apps/argocd-notifications-controller-59f6949d7    1         1         1       2m26s
  replicaset.apps/argocd-redis-75c946f559                      1         1         1       2m26s
  replicaset.apps/argocd-repo-server-6959c47c44                1         1         1       2m26s
  replicaset.apps/argocd-server-65544f4864                     1         1         1       2m26s
  
  NAME                                             READY   AGE
  statefulset.apps/argocd-application-controller   1/1     2m26s
  ```
  - Make ArgoCD accessible over load balancer address for consistent public endpoint accessibility
    
#### ArgoCD Configuration:
#### Important Points OR Advantages over other CI tools:
- ArgoCD solves the security concern by running inside Kubernetes cluster and pulling the manifest out of GitHub repository.
- ArgoCD supports:
  - Kubernetes YAML, Helm Charts, Kustomize and other template files which generates the k8s manifests.
- ArgoCD watches the changes from cluster as well as the Git repository and keep them at desired configuration similar to what in Git repository:
  - ArgoCD is built specifically for GitOps, treating Git as the single source of truth
  - Automatically detects and synchronizes the desired state from Git with the actual state in the cluster
- ArgoCD performs auto rollback
- ArgoCD can help to quickly build the infrastructure during the disaster or region failure situations.
- K8S access control via Git - If you limit the user access at Git level those user will be able to trigger the ArgoCD sync changes to cluster.
