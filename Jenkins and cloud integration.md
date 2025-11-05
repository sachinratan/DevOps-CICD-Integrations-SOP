Following are steps to integrate the Jenkins master with cloud provider e.g. Amazon EKS

Pre-requisites : Install required plugin

Go to path `Jenkins >> Manage Jenkins >> Plugins` and install following plugin which will be required for cloud, pipeline, and docker cliemt up.

- Kubernetes plugin
- Pipeline plugin
- Git plugin
- Docker plugin

Part 1: Configure self k8s cluster for deployment where Jenkins master running on same k8s cluster:

Step 1 : Configure Cloud:

Go to path `Jenkins >> Manage Jenkins >> Clouds >> New cloud` and give the name to cloud:

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_cloud.png)

2.a: Configure the cloud with following kubernetes setting:
```
Kubernetes settings:
- Name: kubernetes
- Kubernetes URL: https://kubernetes.default.svc
- Kubernetes Namespace: jenkins
- Jenkins URL: http://jenkins.jenkins.svc.cluster.local
- Jenkins tunnel: jenkins.jenkins.svc.cluster.local:50000
```

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_cloud_1.png)

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_cloud_3.png)

Step 2 : Configure the pod template (optional):

Go to `Jenkins >> Manage Jenkins >> Clouds >> select the configured cloud name (Amazon_EKS) >> Select pod template >> New pod template`

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_pod_template.png)

You can configure jenkins agent pod template with following pod configurations

![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_pod_template_config.png)
![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_template_config_1.png)
![alt text](https://github.com/sachinratan/k8s-dkr-cicd/blob/main/miscellaneous-data/jnks_add_pod_template_config_2.png)

Part 2: Configure external k8s cluster for deployment where Jenkins master running running outside the k8s cluster:

Step 1 : Configure Cloud:

Go to path `Jenkins >> Manage Jenkins >> Clouds >> New cloud` and give the name to cloud:


