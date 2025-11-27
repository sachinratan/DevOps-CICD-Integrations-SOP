## GitLab CICD Setup and Kubernetes Integration n Deployment

### Method 1:

#### Deploying the GitLab runner on EKS cluster using `helm install` command.
##### Step 1: Add GitLab Helm repository
```
helm repo add gitlab https://charts.gitlab.io
```
##### Step 2: Get the GitLab runner registration token from the GitLab.
- For Project-Specific Runners:
  - Navigate to your GitLab project
  - Go to Settings > CI/CD
  - Expand the Runners section
  - You'll find the registration token there
  - Refer this snip
  ![](https://github.com/sachinratan/DevOps-CICD-Integrations-SOP/blob/main/miscellaneous-data/Runner_reg-token.png)

##### Step 3: Install GitLab Runner
```
helm install gitlab-runner gitlab/gitlab-runner \
  --namespace gitlab-runner \
  --create-namespace \
  --set runnerRegistrationToken="YOUR_TOKEN" \
  --set gitlabUrl="https://gitlab.com/"
```

##### Step 4: Verify the runner pod status on EKS cluster and assigned project runner section.
- GitLab runner pod status
```
$ kubectl get po -n gitlab-runner
NAME                             READY   STATUS    RESTARTS   AGE
gitlab-runner-745cb95bc4-jl72g   1/1     Running   0          4m47s
```
- GitLab runner status in GitLab Project:

![](https://github.com/sachinratan/DevOps-CICD-Integrations-SOP/blob/main/miscellaneous-data/gitlab-runner.png)

### Method 2:

#### Deploying GitLab runner using custom `values.yaml` file.

##### Step 1: Create a Runner in GitLab UI
- First, create a runner in GitLab to obtain the authentication token:

  - Navigate to your GitLab project/group
  - Go to Settings > CI/CD > Runners
  - Click "New project runner" (or "New group runner")
  - Configure the runner settings:
    - Add tags (e.g., kubernetes, eks)
    - Check "Run untagged jobs" if needed
    - Set other preferences
  - Click "Create runner"
  - Copy the authentication token (starts with glrt-)
  - Snippet for reference

    ![](https://github.com/sachinratan/DevOps-CICD-Integrations-SOP/blob/main/miscellaneous-data/register-runner.png)

##### Step 2: Updated Helm Installation
- Create a values.yaml file with the new token format:
  ```
  ## GitLab Runner Configuration
  gitlabUrl: https://gitlab.com/
  
  ## Use runnerToken instead of runnerRegistrationToken
  runnerToken: "glrt-YOUR_AUTHENTICATION_TOKEN_HERE"
  
  ## RBAC Configuration
  rbac:
    create: true
    rules:
      - apiGroups: [""]
        resources: ["pods", "pods/exec", "pods/log", "pods/attach"]
        verbs: ["get", "list", "watch", "create", "delete", "update", "patch"]
      - apiGroups: [""]
        resources: ["secrets"]
        verbs: ["get", "list", "watch", "create", "delete", "update", "patch"]
      - apiGroups: [""]
        resources: ["configmaps"]
        verbs: ["get", "list", "watch", "create", "delete", "update", "patch"]
      - apiGroups: [""]
        resources: ["services"]
        verbs: ["get", "list", "watch", "create", "delete"]
  
  ## Runner Configuration
  runners:
    config: |
      [[runners]]
        [runners.kubernetes]
          namespace = "{{.Release.Namespace}}"
          image = "ubuntu:22.04"
          privileged = false
  ```
##### Step 3: Install with Helm
- Add GitLab Helm repository
```
helm repo add gitlab https://charts.gitlab.io
helm repo update
```
- Install GitLab Runner
```
helm install gitlab-runner gitlab/gitlab-runner \
  --namespace gitlab-runner \
  --create-namespace \
  -f values.yaml
```

##### Step 4: Verify the runner pod status on EKS cluster and assigned project runner section.
- GitLab runner pod status
```
$ kubectl get po -n gitlab-runner
NAME                             READY   STATUS    RESTARTS   AGE
gitlab-runner-64f48c794c-6dbsf   1/1     Running   0          10m
```
- GitLab runner status in GitLab Project:
  ![](https://github.com/sachinratan/DevOps-CICD-Integrations-SOP/blob/main/miscellaneous-data/register-runner-eks.png)
