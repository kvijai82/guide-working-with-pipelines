---
permalink: /guides/pipelines-getting-started/
layout: guide-markdown
title: Build and deploy applications with pipelines
duration: 30 minutes
releasedate: 2020-02-19
description: Explore how to use Pipelines with Application Stacks
tags: ['Pipelines, Application stacks']
guide-category: pipelines
---

<!-- Note:
> This repository contains the guide documentation source. To view
> the guide in published form, view it on the [website](https://kabanero.io/guides/{projectid}.html).
-->

<!--
//
//	Copyright 2019, 2020 IBM Corporation and others.
//
//	Licensed under the Apache License, Version 2.0 (the "License");
//	you may not use this file except in compliance with the License.
//	You may obtain a copy of the License at
//
//	http://www.apache.org/licenses/LICENSE-2.0
//
//	Unless required by applicable law or agreed to in writing, software
//	distributed under the License is distributed on an "AS IS" BASIS,
//	WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
//	See the License for the specific language governing permissions and
//	limitations under the License.
//
-->

Kabanero uses [pipelines](https://github.com/tektoncd/pipeline/tree/master/docs#usage) to illustrate a continuous input and continuous delivery (CI/CD) workflow. Kabanero provides a set of default tasks and pipelines that can be associated with application stacks.  These pipelines validate the application stack is active, build the application stack, publish the image to a container registry, scan the published image, and then deploy the application to the Kubernetes cluster. You can also create your own tasks and pipelines and customize the pre-built pipelines and tasks. All tasks and pipelines are activated by  [Kabanero's standard Kubernetes operator](https://github.com/kabanero-io/kabanero-operator).

To learn more about pipelines and creating new tasks, see [the pipeline tutorial](https://github.com/tektoncd/pipeline/blob/master/docs/tutorial.md).

## Default tasks and pipelines

The default Kabanero tasks and pipelines are provided in the [Kabanero pipelines repository](https://github.com/kabanero-io/kabanero-pipelines/tree/master/pipelines/incubator).  These can be associated with an application stack in the Kabanero custom resource definition (CRD). This is an example CRD:

```yaml
apiVersion: kabanero.io/v1alpha1
kind: Kabanero
metadata:
  name: kabanero
spec:
  version: "0.6.0"
  stacks:
    repositories:
    - name: central
      https:
        url: https://github.com/kabanero-io/collections/releases/download/0.5.0/kabanero-index.yaml
    pipelines:
    - id: default
      sha256: 14d59b7ebae113c18fb815c2ccfd8a846c5fbf91d926ae92e0017ca5caf67c95
      https:
        url: https://github.com/kabanero-io/kabanero-pipelines/releases/download/0.6.0/default-kabanero-pipelines.tar.gz
```

When the Kabanero operator activates the CRD, it associates the pipelines in the pipelines archive with each of the stacks in the stack hub.  The default pipelines are intended to work with all the stacks in the stack hub in the previous example. All of the pipeline-related resources (such as the tasks, trigger bindings, and pipelines) prefix the name of the resource with the keyword `StackId`.  When the operator activates these resources, it replaces the keyword with the name of the stack it is activating.

### Updating default tasks and pipelines

The default tasks and pipelines can be updated by forking the Kabanero Pipelines repo and editing the files under `pipelines/incubator`.  The easiest way to generate the archive for use by the Kabanero CRD is to run the [package.sh](https://github.com/kabanero-io/kabanero-pipelines/blob/master/ci/package.sh) script. The script generates the archive file with the necessary pipeline artifacts and a `manifest.yaml` file that describes the contents of the archive. Alternatively, you can run the Travis build against a release of your pipelines repo, which also generates an archive file and a `manifest.yaml` file.

<!--
// =================================================================================================
// Creating new tasks and pipelines
// =================================================================================================
-->

## Creating new tasks and pipelines

### The build, push and deploy pipeline

- [build-deploy-pl.yaml](https://github.com/kabanero-io/kabanero-pipelines/blob/master/pipelines/incubator/build-deploy-pl.yaml)

  This file is the primary pipeline that showcases all the tasks supplied in the Kabanero repo. It validates that the application stack is active, builds the application stack, publishes the application image to the container registry, does a security scan of the image, and conditionally deploys the application. When running the pipeline via a webhook, the pipeline leverages the triggers functionality to conditionally deploy the application only when a pull request is merged in the git repo.  Other actions that trigger the pipeline run, will validate, build, push, and scan the image.

### Tasks

- [build-task.yaml](https://github.com/kabanero-io/kabanero-pipelines/blob/master/pipelines/incubator/build-task.yaml)

   This file builds a container image from the artifacts in the git-source repository by using [Buildah](https://github.com/containers/buildah). After the image is built, the `build-task` file publishes it to the docker-image URL by using Buildah.

- [deploy-task.yaml](https://github.com/kabanero-io/kabanero-pipelines/blob/master/pipelines/incubator/deploy-task.yaml)

   This file modifies the `app-deploy.yaml` file, which describes the deployment options for the application. `Deploy-task` modifies `app-deploy.yaml` to point to the image that was published and deploys the application by using the application deployment operator. Generate your `app-deploy.yaml` file by running the `appsody deploy --generate-only` command.

By default, the pipelines run and deploy the application in the `kabanero` namespace. If you want to deploy the application in a different namespace, update the `app-deploy.yaml` file to point to that namespace.

For more information, see [the kabanero-pipelines repo](https://github.com/kabanero-io/kabanero-pipelines).

<!--
// =================================================================================================
// Running pipelines
// =================================================================================================
-->

## Running pipelines

Explore how to use pipelines to build and manage application stacks.

### Prerequisites

1. [Kabanero foundation](https://github.com/kabanero-io/kabanero-foundation) must be installed on a supported Kubernetes deployment.

1. [A pipelines dashboard](https://github.com/tektoncd/dashboard) is installed by default with Kabanero's Kubernetes operator. To find the pipelines dashboard URL, login to your cluster and run the `oc get routes` command or in the Kabanero landing page.

1. A persistent volume must be configured. See the following section for details.

1. Secrets for the git repo (if private) and image repository

### Getting started

#### Setting up a persistent volume to run pipelines

Pipelines require a configured volume that is used by the framework to share data across tasks.  The pipeline run creates a Persistent Volume Claim (PVC) with a requirement for five GB of persistent volume.

##### Static persistent volumes

If you are not running your cluster on a public cloud, you can set up a static persistent volume using NFS. For an example of how to use static persistent volume provisioning, see [Static persistent volumes](https://github.com/kabanero-io/kabanero-pipelines/blob/master/docs/VolumeProvisioning.md#static-persistent-volumes).

##### Dynamic volume provisioning

If you run your cluster on a public cloud, you can set up a dynamic persistent volume by using your cloud provider’s default storage class. For an example of how to use dynamic persistent volume provisioning, see [Dynamic volume provisioning](https://github.com/kabanero-io/kabanero-pipelines/blob/master/docs/VolumeProvisioning.md#dynamic-volume-provisioning).

#### Creating secrets

Git secrets must be created in the `kabanero` namespace and associated with the service account that runs the pipelines. To configure secrets using the pipelines dashboard, see [Create secrets](https://kabanero.io/docs/ref/general/configuration/tekton-webhooks.html#create-secrets).

Alternatively, you can [configure secrets in the Kubernetes console or set them up by using the Kubernetes CLI](https://docs.okd.io/latest/dev_guide/secrets.html#creating-secrets).

### Running pipelines by using the pipelines dashboard webhook extension

You can use the [pipelines dashboard webhook extension](https://github.com/tektoncd/experimental/blob/master/webhooks-extension/docs/GettingStarted.md) to drive pipelines that automatically build and deploy an application whenever you update the code in your Git repo. Events such as commits or pull requests can be set up to automatically trigger pipeline runs.

### Running pipelines by using a script

If you are developing a new pipeline and want to test it in a tight loop, you might want to use a script or manually drive the pipeline.

1. Log in to your cluster. For example,

   ```shell
   oc login <master node IP>:8443
   ```

1. Clone the pipelines repo

   ```shell
   git clone https://github.com/kabanero-io/kabanero-pipelines
   ```

1. Run the following script with the appropriate parameters

   ```shell
   cd ./pipelines/sample-helper-files/./manual-pipeline-run-script.sh -r [git_repo of the Appsody project] -i [docker registery path of the image to be created] -c [application stack name of which pipeline to be run]"
   ```

   - The following example is configured to use the dockerhub container registry:

      ```shell
       ./manual-pipeline-run-script.sh -r https://github.com/mygitid/appsody-test-project -i index.docker.io/mydockeid/my-java-microprofile-image -c java-microprofile"
      ```

   - The following example is configured to use the local OpenShift container registry:

      ```shell
       ./manual-pipeline-run-script.sh -r https://github.com/mygitid/appsody-test-project -i docker-registry.default.svc:5000/kabanero/my-java-microprofile-image -c java-microprofile"
      ```

### Running pipelines manually from the command line

1. Login to your cluster. For example,

   ```shell
   oc login <master node IP>:8443
   ```

1. Clone the pipelines repo.

   ```shell
   git clone https://github.com/kabanero-io/kabanero-pipelines
   cd kabanero-pipelines
   ```

1. Create pipeline resources.

   Use the `pipeline-resource-template.yaml` file to create the `PipelineResources`. The `pipeline-resource-template.yaml` is provided in the pipelines [/pipelines/sample-helper-files](https://github.com/kabanero-io/kabanero-pipelines/tree/master/pipelines/sample-helper-files) directory. Update the docker-image URL. You can use the sample GitHub repo or update it to point to your own GitHub repo.

1. After you update the file, apply it as shown in the following example:

   ```shell
   oc apply -f <stack-name>-pipeline-resources.yaml
   ```

### Activating tasks and pipelines

The installations that activate the featured application stacks also activate the tasks and pipelines. If you are creating a new task or pipeline, activate it manually, as shown in the following example.

```shell
oc apply -f <task.yaml>
oc apply -f <pipeline.yaml>
```

### Running the pipeline

A sample `manual-pipeline-run-template.yaml` file is provided in the [/pipelines/sample-helper-files](https://github.com/kabanero-io/kabanero-pipelines/tree/master/pipelines/sample-helper-files) directory. Rename the template file to a name of your choice (for example, pipeline-run.yaml), and update the file to replace `application-stack-name` with the name of your application stack. After you update the file, run it as shown in the following example.

```shell
oc apply -f <application-stack-name>-pipeline-run.yaml
```

<!--
// =================================================================================================
// Running pipelines from the command line for your custom built application stacks
// =================================================================================================
-->

## Running pipelines from the command line for your custom built application stacks

The following steps explain how to run pipelines against custom built application stack images instead of the provided application stacks.

### Setting up a container registry URL for the custom application stack image

By default, pipelines pull the application stack images from Docker hub. If you are publishing your application stack images to any other registry, use the following process to configure the custom repository from which your pipelines pull the container images.

1. After you clone the `kabanero-pipelines` repository, find the `stack-image-registry-map.yaml` configmap template file. Add your container registry URL to this file in place of the `default-stack-image-registry-url` statement.

   ```shell
   cd kabanero-pipelines/pipelines/common/
   vi stack-image-registry-map.yaml
   ```

1. Apply the following configmap file, which will set your container registry.

   ```shell
   oc apply -f stack-image-registry-map.yaml
   ```

#### Setting up a container registry URL for a custom application stack image that is stored in a container registry with an internal route URL on the cluster

For an internal OpenShift registry, set up the `stack-image-registry-map.yaml` file with the internal registry URL.

NOTE : In this case, the service account that is associated with the pipelines must be configured to allow the pipelines pull from the internal registry without configuring a secret.

#### Setting up a container registry URL for a custom application stack image that is stored in a container registry with an external route URL

For a container image with an external container registry route URL, you must set up a Kubernetes secret. To set up this secret, update the `default-stack-image-registry-secret.yaml` template file with a Base64 formatted username and password and apply it to the cluster, as described in the following steps.

1. First, update the `stack-image-registry-map.yaml` file with your container registry file, as described in step 1 of `Set up a container registry URL for the custom application stack image`.

1. Find the `default-stack-image-registry-secret.yaml` template file in the cloned kabanero-pipelines repo (`kabanero-pipelines/pipelines/common`) and update it with the username and token password for the container registry URL you specified previously.

1. Create a Base64 format version of the username and password for the external route container registry URL.

   ```shell
   echo -n <your-registry-username> | base64
   echo -n <your-registry-password> | base64
   ```

1. Update the `default-stack-image-registry-secret.yaml` file with the Base64 formatted username and password.

   ```shell
   vi default-stack-image-registry-secret.yaml
   ```

1. Apply the `default-stack-image-registry-secret.yaml` file to the cluster

   ```shell
   oc apply -f default-stack-image-registry-secret.yaml
   ```

1. You can now run the pipeline by following the steps in the preceding `Run pipelines from the command line for your custom built application stacks` section.

<!--
// =================================================================================================
// Checking the status of the pipeline run
// =================================================================================================
-->

## Checking the status of the pipeline run

You can check the status of the pipeline run from the Kubernetes console,
command line, or pipelines dashboard.

### Checking pipeline run status from the pipelines dashboard

1. Log in to the pipelines dashboard and click `Pipeline runs'
in the sidebar menu.

1. Find your pipeline run in the list and click it to check the status and find logs. You can see logs
and status for each step and task.

### Checking pipeline run status from the command line

Enter the following command in the terminal:

```shell
oc get pipelineruns
oc -n kabanero describe pipelinerun.tekton.dev/<pipeline-run-name>
```

You can also see pods for the pipeline runs, for which you can specify `oc describe` and `oc logs` to get more details.

If the pipeline run was successful, you can see a Docker image in our Docker registry and a pod that’s running your application.

<!--
// =================================================================================================
// Troubleshooting
// =================================================================================================
-->

## Troubleshooting

To find solutions for common issues and troubleshoot problems with pipelines, see the [Pipelines Troubleshooting Guide](https://github.com/kabanero-io/kabanero-pipelines/blob/master/docs/Troubleshooting.md).

### Related links

- [Kabanero Pipelines repository](https://github.com/kabanero-io/kabanero-pipelines)
- [Tekton Pipeline tutorial](https://github.com/tektoncd/pipeline/blob/master/docs/tutorial.md)