## GitLab CICD Setup and Kubernetes Integration n Deployment

### Deploying the GitLab runner on EKS cluster.
#### Step 1: Add GitLab Helm repository
```
helm repo add gitlab https://charts.gitlab.io
```
#### Step 2: Get the GitLab runner registration token from the GitLab.
- For Project-Specific Runners:
  - Navigate to your GitLab project
  - Go to Settings > CI/CD
  - Expand the Runners section
  - You'll find the registration token there
  - Refer this snip
  ![](https://github.com/sachinratan/DevOps-CICD-Integrations-SOP/blob/main/miscellaneous-data/Runner_reg-token.png)

#### Step 3: Install GitLab Runner
```
helm install gitlab-runner gitlab/gitlab-runner \
  --namespace gitlab-runner \
  --create-namespace \
  --set runnerRegistrationToken="YOUR_TOKEN" \
  --set gitlabUrl="https://gitlab.com/"
```

#### Step 4: Verify the runner pod status on EKS cluster and assigned project runner section.
- GitLab runner pod status
```
$ kubectl get po -n gitlab-runner
NAME                             READY   STATUS    RESTARTS   AGE
gitlab-runner-745cb95bc4-jl72g   1/1     Running   0          4m47s
```
- GitLab runner status in GitLab Project:

![](https://github.com/sachinratan/DevOps-CICD-Integrations-SOP/blob/main/miscellaneous-data/gitlab-runner.png)

