# Jenkins CI/CD Pipeline for Weather Application

This repository contains the Jenkins pipeline for building, pushing Docker images, and deploying to Kubernetes using ArgoCD. The pipeline automates the following steps:

1. **Building the Docker image**
2. **Pushing the Docker image to Docker Hub**
3. **Authenticating with ArgoCD**
4. **Creating or Updating an application in ArgoCD**
5. **Synchronizing the application in ArgoCD**
6. **Sending notifications to Slack**

## Prerequisites

Before running the pipeline, ensure the following:

- **Docker** is installed on the Jenkins agent for building and pushing images.
- **ArgoCD** is set up and accessible on the `ARGOCD_SERVER` (default is `localhost:8082`).
- **Docker Hub credentials** are stored in Jenkins under the credential ID `dockerhub`.
- **ArgoCD credentials** are stored in Jenkins under the credential ID `argocdcred`.
- A **Slack webhook** and credentials should be configured in Jenkins to send build notifications.

## Pipeline Overview

This Jenkins pipeline contains the following stages:

### 1. Build Image
- **Description**: Builds a Docker image from the `Dockerfile` in the repository.
- **Commands**:
  - `docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .`
- The image is tagged with the build number for versioning.

### 2. Push Image
- **Description**: Logs into Docker Hub and pushes the Docker image.
- **Credentials**: The pipeline uses Jenkins credentials for Docker Hub (`dockerhub`).
- **Commands**:
  - `docker login --username $USERNAME --password $PASSWORD`
  - `docker push ${IMAGE_NAME}:${BUILD_NUMBER}`

### 3. Authenticate with ArgoCD
- **Description**: Authenticates with ArgoCD using the provided credentials.
- **Credentials**: ArgoCD login credentials are retrieved from Jenkins (`argocdcred`).
- **Commands**:
  - `argocd login $ARGOCD_SERVER --username $username --password $password --insecure`

### 4. Create App in ArgoCD
- **Description**: Creates a new ArgoCD application that uses the newly built Docker image.
- **Commands**:
  - `argocd app create $APP_NAME \
     --repo https://github.com/muhammedhamedelgaml/weather_app_argocd_helm.git\
     --path helm/weathercharts \
     --helm-set image=${IMAGE_NAME}:${BUILD_NUMBER} \
     --dest-server https://kubernetes.default.svc \
     --dest-namespace default
`
  - This includes specifying the Helm chart repository, path to the chart, the image, and the Kubernetes namespace.

### 5. Resync the Application
- **Description**: Syncs the ArgoCD application to apply the changes.
- **Commands**:
  - `argocd app sync $APP_NAME`

### 6. Post Actions
- **Description**: Cleanup and notifications.
- **Actions**:
  - Logs out from Docker.
  - Sends a Slack notification about the build status.

## Environment Variables

The following environment variables are used in the pipeline:

- `ARGOCD_SERVER`: The address of the ArgoCD server (default: `localhost:8082`).
- `APP_NAME`: The name of the application in ArgoCD (default: `weather`).
- `IMAGE_NAME`: The Docker image name (default: `muhammedhamedelgaml/app_python`).

## Notifications

The pipeline sends notifications to a Slack channel (`#cicdjenkins`) to inform about the build status. The color of the notification depends on the build result:

- `SUCCESS`: Green
- `FAILURE`: Red

## Credentials

The pipeline uses the following credentials stored in Jenkins:

1. **Docker Hub Credentials** (`dockerhub`): Used for logging into Docker Hub and pushing the image.
2. **ArgoCD Credentials** (`argocdcred`): Used for logging into ArgoCD.

Ensure these credentials are correctly set up in Jenkins before running the pipeline.

## Slack Notification Configuration

A Slack notification will be sent after the pipeline run to report the build status. The message includes the following details:

- Job status (success or failure)
- Build URL for more information

Example of a notification:
![SLACK ](/screenshots/slack.png)

## Argocd screenshot
![Argocd](/screenshots/argocd.png)


ee
