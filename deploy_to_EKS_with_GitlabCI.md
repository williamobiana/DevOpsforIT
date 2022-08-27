# Deploy a simple application on AWS EKS with Gitlab CI

The goal of this article is to show how you can automate the build and deploy of your containerized application directly from Gitlab to AWS EKS.

## Prerequisites:
* a running AWS EKS cluster with `kubeconfig` file (to deploy an EKS cluster, checkout the aws guide [here](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html))
* an active Gitlab runner
* an active Gitlab container registry for images

## Steps:
* configure Gitlab CI/CD variables in the project settings to connect to the AWS account and EKS cluster
* give the EKS cluster credentials to access the Gitlab container registry
* prepare the deployment manifest
* write the build and deploy jobs in the .gitlab-ci.yml file
* push to test your build.

### Configure Gitlab CI/CD variables
For security reasons on Gitlab, it is recommended to store your sensitive informations such as password, API keys, files,...etc, in the CI/CD variables of the project settings, saved as key:value pairs.

You can reference the varaible keys in your gitlab-ci.yml file, this prevents your information from becoming exposed to the public.

Most CI/CD variables are predefined by default in Gitlab: https://docs.gitlab.com/ce/ci/variables/

To create a new variable, you must have at least a maintainer role.\
go to the gitlab project: `Settings > CI/CD > Variables`

add 3 `variables` types:
* `AWS_ACCESS_KEY_ID` = Access ID of your AWS account
* `AWS_SECRET_ACCESS_KEY` = Secret key of your AWS account
* `AWS_DEFAULT_REGION` = AWS region where your EKS cluster is running

add a `file` type:
* `KUBECONFIG` = paste your EKS cluster kubeconfig 

### Give EKS access to Gitlab container registry
This will be divided into 2 micro steps:
* create a docker-credential file
* create a kubernetes secret

To pull an image from a private registry, you need to have `docker` in your CLI to logging to the Gitlab private registry.

To allow the EKS cluster pull images from the container registry, we will give it the login credentials for gitlab.
This process is explained in the docker documentation [here](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

First we login into the gitlab container registry using docker
```
docker login registry.gitlab.com -u USERNAME -p PASSWORD
```

The login process creates or updates a `config.json` file that holds an authorization token.\
view the `config.json` file  
```
cat ~/.docker/config.json
```
copy the section of the output containing the auths for the registry like this:
```
{
    "auths": {
        "YOUR.GITLAB.REGISTRY:1234": {
            "auth": "jhbG5ug7GJ8mJJVm1KMnduaJslUDutDF6GDH"
        }
    }
}
```
#### Create a docker-credential file
create the `docker-cred.json` file and paste the auths.

Note: The “auth” is the base64 encoded credentials generated with the conbination of the gitlab username:password.
```
echo "gitlab_username:gitlab_password" | base64
```

after we create the `docker-cred.json` file, encode it in base64. This will be used to create a kubernetes secret file.
```
cat docker-creds.json | base64
```

#### Create a kubernetes secret
create the `gitlab-registry.yaml` file as seen below:
```
apiVersion: v1
kind: Secret
metadata:
  name: gitlab
  namespace: default
data:
  .dockerconfigjson: DjdlkDu7hD75UhDkpjkdkjdjgvxvjdmoJnaXRsYWIuYXVkaXZveC5mcjo0NTY3IjogewogICAgICAgICAgICAgICAgICAiYXV0aC077Jh36ggdysDJjhd322bdcjDhTW5kdWEyTkVURnBuZDIxSE5tdEgiCiAgICAgICAgICB9CiHFvyI87vD5hnMcnMiOiB7CiAgICAgICAgICAiVXNlci1BZ2VudCI6ICJEbZXKuHnnSJDOL5hD4LjEtY2UgKGxpbnV4KSIKICB9Cn0K
type: kubernetes.io/dockerconfigjson
```
Replace the `.dockerconfigjson:` long random string with the `base64 encoded docker-creds.json` string

Apply the the file to our kubernetes secret.
```
kubectl apply -f secret-registry.yaml
```

### Prepare the deployment manifest
In our deployment manifest called `deployment.yaml`, we have to reference 2 key information
* the proper `image name` from our registry
* an `image pull policy` to allow access to the registry using our secret

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-app
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: app
        image: YOUR_GITLAB_REGISTRY.DOMAIN.TLD:1234/api:<VERSION>
        ports:
        - containerPort: 80
      imagePullSecrets:
        - name: gitlab   #referenced from secret-registry.yaml
```

### Write the gitlab-ci.yml yaml 
In our gitlab-ci.yml file, we define the following steps:
* build image and push to gitlab registry
* deploy to the kubernetes cluster

```
stages:
  - build
  - deploy

buildJob:
  image: docker:latest
  stage: build
  before_script:
    - docker login ${CI_REGISTRY} -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASSWORD} 
  script:
    - docker build --file Dockerfile-CI -t ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHORT_SHA} .
    - docker tag ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHORT_SHA} ${CI_REGISTRY_IMAGE}:latest
    - docker push ${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHORT_SHA}
    - docker push ${CI_REGISTRY_IMAGE}:latest

deployJob:
  image: tfgco/kubectl
  stage: deploy
  before_script:
    - export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
    - export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
    - export AWS_DEFAULT_REGION=${AWS_DEFAULT_REGION}
  script:
    - kubectl --kubeconfig ${KUBECONFIG} apply -f deployment.yaml
```

### Push and test
Now we have out process complete, we can proceed to git add, commit and push to our branch.\
We can verify the deployment in our cluster via the CLI or the aws webUI.
Depending on your needs, you could add jobs/stages to the CI and modify the manifest.
