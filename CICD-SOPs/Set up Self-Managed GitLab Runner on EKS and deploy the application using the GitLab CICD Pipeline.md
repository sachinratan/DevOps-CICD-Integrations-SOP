## Set up Self-Managed GitLab Runner on EKS and deploy the application using the GitLab CI/CD Pipeline

#### Pre-requisites: Follow the step of instruction from the [GitLab CICD Setup and Kubernetes Integration n Deployment](https://github.com/sachinratan/DevOps-CICD-Integrations-SOP/blob/main/CICD-SOPs/GitLab%20CICD%20Setup%20and%20Kubernetes%20Integration%20n%20Deployment.md#gitlab-cicd-setup-and-kubernetes-integration-n-deployment) Method 2 - [Deploying GitLab runner using custom values.yaml file](https://github.com/sachinratan/DevOps-CICD-Integrations-SOP/blob/main/CICD-SOPs/GitLab%20CICD%20Setup%20and%20Kubernetes%20Integration%20n%20Deployment.md#deploying-gitlab-runner-using-custom-valuesyaml-file)

#### Step 1: Update the `value.yaml` similar to following configuration.
```
cat values.yaml
## GitLab Runner Configuration
gitlabUrl: https://gitlab.com/

## Use runnerToken instead of runnerRegistrationToken
runnerToken: "glrt-xxxx.01.1j1g27q0j"

## RBAC Configuration
rbac:
  create: true
  serviceAccountName: gitlab-runner
  serviceAccountAnnotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<AWS_ACC_NO>:role/GitLabRunnerECRRole
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
        privileged = true
        service_account = "gitlab-runner"

        # CRITICAL: Configure shared volumes for DinD certificates
        [[runners.kubernetes.volumes.empty_dir]]
          name = "docker-certs-client"
          mount_path = "/certs/client"
          medium = "Memory"
```
#### Step 2: Deploy the changes to GitLab runner.
```
helm upgrade gitlab-runner gitlab/gitlab-runner --namespace gitlab-runner -f values.yaml
```

#### Step 3: Create the sample application.
- You can use the sample GO app from - https://gitlab.com/slrs-admin/go-curd-app.git

#### Step 4: Write the GitLab CICD pipeline `.gitlab-ci.yml` file.
```
# This file is a template, and might need editing before it works on your project.
# This is a sample GitLab CI/CD configuration file that should run without any modifications.
# It demonstrates a basic 3 stage CI/CD pipeline. Instead of real tests or scripts,
# it uses echo commands to simulate the pipeline execution.
#
# A pipeline is composed of independent jobs that run scripts, grouped into stages.
# Stages run in sequential order, but jobs within stages run in parallel.
#
# For more information, see: https://docs.gitlab.com/ee/ci/yaml/#stages
#
# You can copy and paste this template into a new `.gitlab-ci.yml` file.
# You should not add this template to an existing `.gitlab-ci.yml` file by using the `include:` keyword.
#
# To contribute improvements to CI/CD templates, please follow the Development guide at:
# https://docs.gitlab.com/development/cicd/templates/
# This specific template is located at:
# https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/Getting-Started.gitlab-ci.yml
variables:
  AWS_DEFAULT_REGION: eu-central-1
  ECR_REGISTRY: <AWS_ACC_NO>.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  ECR_REPOSITORY: gitlab-ci-pipeline-image-repo
  EKS_CLUSTER_NAME: sandbox-env-cluster
  K8S_NAMESPACE: default

stages:          # List of stages for jobs, and their order of execution
  - test
  - build
  - push-test
  - push
  - deploy

unit-test-job:   # This job runs in the test stage.
  stage: test 
  image: golang:1.20-alpine   # It only starts when the job in the build stage completes successfully.
  tags:
    - aws_eks_runner
  script:
    - echo "Running unit tests... This will take about 60 seconds."
    - go test ./...
    - echo "Code coverage is 90%"

build-job:       # This job runs in the build stage, which runs first.
  stage: build
  image: golang:1.20-alpine   # It only starts when the job in the build stage completes successfully.
  tags:
    - aws_eks_runner
  script:
    - echo "Compiling the code..."
    - CGO_ENABLED=0 GOOS=linux go build -o go-curd-app .
    - echo "Compile complete."

push-job:
  stage: push
  image: docker:24.0.7
  services:
    - name: docker:24.0.7-dind
      alias: docker
      #command: ["--tls=false"]
  variables:
   DOCKER_HOST: tcp://docker:2376
   DOCKER_TLS_CERTDIR: "/certs"
   DOCKER_TLS_VERIFY: 1
   DOCKER_CERT_PATH: "$DOCKER_TLS_CERTDIR/client"
  before_script:
    # No AWS credentials needed! IRSA provides them automatically
    - apk add --no-cache aws-cli

    # Verify CLI installation
    - aws --version

    # Verify the IRSA role credentials
    - aws sts get-caller-identity

    # Wait for Docker daemon to be ready
    #- echo "Waiting for Docker daemon to start..."
    #- timeout 360 sh -c 'until docker info >/dev/null 2>&1; do echo "Waiting for docker daemon..."; sleep 2; done'
    #- echo "Docker daemon is ready!"
    - sleep 30

    # Verify Docker is working
    - docker info
    
    # The AWS CLI automatically uses the IRSA credentials
    - aws ecr get-login-password --region $AWS_DEFAULT_REGION| docker login --username AWS --password-stdin $ECR_REGISTRY
  
    # Generate image tag
    - export IMAGE_TAG="${CI_COMMIT_SHORT_SHA}-$(date +%Y%m%d)-${CI_PIPELINE_IID}"
    - echo "Building image with tag is $IMAGE_TAG"

  tags:
    - aws_eks_runner
  script:
    - echo "Building Docker image..."
    - docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
    - docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - echo "Pushing to ECR..."
    - docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
    - docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

    - echo "Image pushed successfully!"
  artifacts:
    reports:
      dotenv: build.env
  after_script:
    # Save the full tag for use in deploy stage
    - echo IMAGE_TAG="${CI_COMMIT_SHORT_SHA}-$(date +%Y%m%d)-${CI_PIPELINE_IID}" >> build.env
  only:
    - main

deploy-to-eks:
  stage: deploy
  image:
    name: amazon/aws-cli:latest
    entrypoint: [""]
  before_script:
    # Install kubectl and gettext (for envsubst)
    - curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    - chmod +x kubectl
    - mv kubectl /usr/local/bin/
    - yum install -y gettext

    # Configure kubectl
    - aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_DEFAULT_REGION

    # Generate the IMAGE URI
    - - export IMAGE_FULL="${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"

    # Confirm the image tag
    - echo "Deploying image ${IMAGE_FULL}"
  script:
    # Create namespace if not using default
    - |
      if [ "${K8S_NAMESPACE}" != "default" ]; then
        echo "Creating namespace ${K8S_NAMESPACE} if it doesn't exist..."
        kubectl create namespace ${K8S_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
      fi

    # Check if deployment exists
    - |
      if kubectl get deployment go-crud-app -n ${K8S_NAMESPACE} >/dev/null 2>&1; then
        echo "✅ Deployment exists. Updating image tag..."
        kubectl set image deployment/go-crud-app \
          go-crud-app=${IMAGE_FULL} \
          -n ${K8S_NAMESPACE}

        # Annotate deployment with update info
        kubectl annotate deployment go-crud-app \
          kubernetes.io/change-cause="Updated to ${IMAGE_FULL} by pipeline ${CI_PIPELINE_IID}" \
          -n ${K8S_NAMESPACE} --overwrite
      else
        echo "⚠️  Deployment does not exist. Creating from manifest..."

        # Substitute environment variables in deployment file
        envsubst < go-crud-app-deployment.yaml | kubectl apply -f - -n ${K8S_NAMESPACE}

        echo "✅ Deployment created successfully"
      fi
    # Wait and verify (same as before)
    - kubectl rollout status deployment/go-crud-app -n ${K8S_NAMESPACE} --timeout=10m
    - kubectl wait --for=condition=ready pod -l app=go-crud-app -n ${K8S_NAMESPACE} --timeout=5m

    # Display deployment information
    - echo "📊 Deployment Summary:"
    - kubectl get deployment go-crud-app -n ${K8S_NAMESPACE}
    - kubectl get pods -n ${K8S_NAMESPACE} -l app=go-crud-app
    - kubectl get svc go-crud-app-service -n ${K8S_NAMESPACE}

    - echo "🎉 Deployment triggered successfully with image ${IMAGE_FULL}"
  only:
    - main
  tags:
    - aws_eks_runner
```

### Optional: To avoiding exposing the cloud credential to GitLab pipeline, leverage the IAM Role for ServiceAccount (IRSA)
#### Step 1: Create IAM Policy for ECR Access:
```
# Create ECR policy file
cat > ecr-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload"
      ],
      "Resource": "arn:aws:ecr:*:<AWS_ACC_NO>:repository/*"
    },
    {
            "Effect": "Allow",
            "Action": [
                "eks:DescribeCluster",
                "eks:ListClusters"
            ],
            "Resource": "arn:aws:eks:*:<AWS_ACC_NO>:cluster/*"
   }
  ]
}
EOF

# Create the policy
aws iam create-policy \
  --policy-name GitLabRunnerECRPolicy \
  --policy-document file://ecr-policy.json
```
#### Step 2: Enable OIDC Provider on EKS (optional: if not done already)
```
# Get your cluster's OIDC issuer URL
OIDC_ISSUER=$(aws eks describe-cluster --name <your-cluster-name> --region us-east-1 --query "cluster.identity.oidc.issuer" --output text)

# If empty, enable OIDC provider
eksctl utils associate-iam-oidc-provider \
  --cluster <your-cluster-name> \
  --region eu-central-1 \
  --approve
```

#### Step 3: Create IAM Role with Trust Relationship
```
# Extract OIDC provider ID
OIDC_PROVIDER=$(echo $OIDC_ISSUER | sed 's/https:\/\///')

# Create trust policy
cat > trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<AWS_ACC_NO>:oidc-provider/${OIDC_PROVIDER}"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "${OIDC_PROVIDER}:sub": "system:serviceaccount:gitlab-runner:gitlab-runner",
          "${OIDC_PROVIDER}:aud": "sts.amazonaws.com"
        }
      }
    }
  ]
}
EOF

# Create the IAM role
aws iam create-role \
  --role-name GitLabRunnerECRRole \
  --assume-role-policy-document file://trust-policy.json

# Attach the ECR policy to the role
aws iam attach-role-policy \
  --role-name GitLabRunnerECRRole \
  --policy-arn arn:aws:iam::<AWS_ACC_NO>:policy/GitLabRunnerECRPolicy
```
#### Step 4: Verify IRSA Configuration
```
# Check service account annotation
kubectl get serviceaccount gitlab-runner -n gitlab-runner -o jsonpath='{.metadata.annotations.eks\.amazonaws\.com/role-arn}'

# Should output: arn:aws:iam::<AWS_ACC_NO>:role/GitLabRunnerECRRole
```
