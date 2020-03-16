# openshift-pipelines-workshop
Workshop to demonstrate OpenShift Pipelines (featuring Tekton).

**All** are welcome: Linux, macOS and Windows.

## Description
This workshop will guide you through the creation, execution and ongoing use of OpenShift Pipelines, a CI/CD instance that runs in an OpenShift cluster. You will learn how OpenShift Pipelines are installed, how they are configured, how they are run, how to handle the results, and how to continue a development cycle using this CI/CD environment.

## Table Of Contents
* [Introduction](#Introduction)
* [Prerequisites](#Prerequisites)
* [Workshop](#Workshop)
* [Optional Steps](#Optional)
* [Conclusion](#Conclusion)


## Introduction
Delivering working code is the goal of software developement. Undelivered code is worthless; we need to get bits into production. It's sometimes easy to lose sight of this given the distractions of new technologies, the latest methodologies, changing frameworks, etc. In the end, however, it's about working code running in production. In this workshop we'll start with a fresh, new OpenShift cluster and finish with an automated CI/CD system -- OpenShift Pipelines -- running in that cluster. Here, in one place, the end-to-end story.

As an optional step, we will then progress to using the automated CI/CD system as we update and push source code.

The engine for this is OpenShift Pipelines. OpenShift Pipelines relies on Tekton, the container-based build component of Knative, and runs in pods. Because each step runs in it's own pod, the benefits include scaling and the ability to run steps in parallel.

This example will feature code written in Java.

## Prerequisites
The following environments and/or tools are necessary to perform this workshop:
1. Terminal on your PC
2. OpenShift version 4.2 cluster
3. Tekton command line interface (CLI), `tkn`
4. OpenShift CLI, `oc`
5. Optional Git repo

### Prerequisite #1: Terminal on your PC
You'll need to be able to run commands at the command line on your PC. Note that this workshop works with Linux and macOS (bash) and Windows (PowerShell). Where any differences occur in this workshop, commands for both environments will be supplied.

### Prerequisite #2: OpenShift cluster
A not-trivial OpenShift 4.2 cluster is necessary to run this workshop. Options include:
* Amazon Web Services (AWS)
* Microsoft Azure
* vmware vSphere
* Bare metal

Visit the [OpenShift 4 web site at try.openshift.com](try.openshift.com) for instructions for your selected infrastructure.

### Prerequisite #3: tkn
The Tekton CLI, `tkn`, is necessary. Note that this utility is built for your operating system of choice.

To install tkn, visit [the Tekton Pipelines cli github repo](https://github.com/tektoncd/cli#getting-started) and follow the instructions for your operating system.

### Prerequisite #4: oc
The OpenShift CLI, `oc`, is necessary. Note that this utility is built for your operating system of choice.

To install oc, visit [the OpenShift version 4 clients download page](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/) and download the client for your operating. You'll need to decompress the downloaded file and make sure the resulting executable bits are in your system PATH variable.

### Prerequisite #5: Git repo
The final section of this workshop will have you updating your existing code that is automatically built and deployed. In order to perform this optional section, you'll need a Git repo from which you will clone your code and into which you will push your code -- which will fire a webhook and start the automated CI/CD pipelines.

If you do not wish to include this section, you will be pulling code from an existing github repo.

## Workshop
The following list shows the seven steps will be used to get our CI/CD pipeline up and running:
1. Create projects
1. Install operator
1. Create subscription
1. Create Pipeline
1. Create Tasks
1. Run the Pipeline
1. View the results

<hr>  
 <div style="background-color: cornsilk; margin-left: 20px; margin-right: 20px">
<h4>Operators and Subscriptions Explained</h4>  

Because OpenShift is built on Kubernetes, it supports the concept of "Operators", or pre-built Customer Resource Descriptions (CRD) that include the sometimes many pieces needed to invoke a solution. In other words, a Kubernetes Operator can be used to start a complex solution. For example, there is a Kubernetes Operator that allows you to very easily get an instance of MongoDB running in your OpenShift cluster. There are others: Eclipse Che, Elasticsearch, Kafka, and dozens more.

To invoke an operator may involve many steps. You install the Operator and the create a Subscription. In some cases, such as Kafka, you then invoke the API you want. For example, Kafka Connect or Kafka Topic.

For OpenShift Pipelines, we need to install the operator and create a subscription. After that we can start building the pieces that comprise our own, specific pipeline.
</div>
<hr>

### Workshop Step 0: Log in
Before any work can begin, you must be logged into your OpenShift cluster with cluster-admin permissions. You can verify this by running the following command:  

`oc get permissions`


### Workshop Step 1: Create projects

`oc new-project pipelines-tutorial`


### Workshop Step 2: Install operator
You have two choices: Use the web UI dashboard or use the command line.  

#### Installing Pipelines operator using the Web UI dashboard

*TODO* (this, the Web UI instruction section, is a work in process)

#### Installing the Pipelines operator using the command line.  

### Set permissions

`oc adm policy add-scc-to-user anyuid -z tekton-pipelines-controller`  

### Workshop Step 3: Create subscription  
 
`oc apply -f sub.yaml`


### Workshop Step 4: Create pipeline  
`oc create -f https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/pipeline/update_deployment_task.yaml`

### Workshop Step 5: Create tasks  

`oc create -f https://raw.githubusercontent.com/openshift/pipelines-tutorial/master/pipeline/apply_manifest_task.yaml`
`tkn clustertask ls`
`tkn task ls`

`tkn resource create`

**TODO** create steps here

`tkn resource ls`

### Step 5.5: Create pipeline

`oc create -f https://raw.githubusercontent/openshift/pipelines-tutorial/master/pipeline/pipeline.yaml`
`tkn pipeline ls`

### Workshop Step 6: Run pipeline  
`tkn pipeline start build-and-deploy`

### Workshop Step 7: View results  

Select the project in your Web UI, choose the Routes option from the menu on the left, and then click the route listed. Your default browser will open with the results.