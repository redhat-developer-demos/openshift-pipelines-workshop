# openshift-pipelines-workshop
## Workshop to demonstrate OpenShift Pipelines (featuring Tekton).

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

As an optional step, we will then move forward, using the automated CI/CD system as we update and push source code. This will complete the cycle of creating source code, updating source code, and updating the compiled bits that are running in an OpenShift pod.

The engine for this is OpenShift Pipelines. OpenShift Pipelines relies on Tekton, the container-based build component of Knative, and runs in pods. Because each step runs in it's own pod, the benefits include scaling and the ability to run steps in parallel.

This example will give you the choice of using code written in Go or C#. A simple RESTful service, Quote Of The Day, will return quotes via an HTTP GET request.

Example:
![tkn resource ls results](images/curl-example.png)

Note: While the screen captures in this document are based on PowerShell, rest assured that everything here works equally well in a bash shell.

## Prerequisites
The following is a list of the environments and/or tools are necessary to perform this workshop. Details about each follow this list.
1. A terminal session on your PC
1. OpenShift version 4.3 (or newer) cluster
1. Tekton command line interface (CLI), `tkn`
1. OpenShift CLI, `oc`
1. Git
1. Git repo at https://github.com/redhat-developer-demos/openshift-pipelines-tutorial.git. You'll need to fork this repo and then clone this repo to your machine.

### Prerequisite #1: A terminal session on your PC
You'll need to be able to run commands at the command line on your PC. Note that this workshop works with Linux and macOS (bash or PowerShell) and Windows (PowerShell). Where any differences between bash and PowerShell occur in this workshop, commands for both environments will be supplied.

*Yes, PowerShell runs on Linux and macOS.*

### Prerequisite #2: OpenShift version 4.3 (or newer) cluster
An OpenShift 4.3 (or newer) cluster is necessary to run this workshop. Options include:
* CodeReady Containers (CRC) **which is zero cost and runs on your local machine**.
* Amazon Web Services (AWS)
* Microsoft Azure
* Google Cloud
* vmware vSphere
* Red Hat OpenStack Platform
* Red Hat Virtualization
* IBM Z IBM LineONE
* Power Systems
* Bare metal

Visit the [OpenShift 4 web site at try.openshift.com](try.openshift.com) for instructions for your selected cloud-based infrastructure.

Visit the [CodeReady Containers web site](https://developers.redhat.com/products/codeready-containers) for instructions regarding CodeReady Containers.

### Prerequisite #3: Tekton command line interface (CLI), `tkn`
The Tekton CLI, `tkn`, is necessary. Note that this utility is built for your operating system of choice.

To install tkn, visit [the Tekton Pipelines cli github repo](https://github.com/tektoncd/cli#getting-started) and follow the instructions for your operating system.

### Prerequisite #4: OpenShift CLI, `oc`
The OpenShift CLI, `oc`, is necessary. Note that this utility is built for your operating system of choice.

To install oc, visit [the OpenShift version 4 clients download page](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/) and download the client for your operating. You'll need to decompress the downloaded file and make sure the resulting executable bits are in your system PATH variable.

### Prerequisite #5: Git
You'll need the Git tool installed on your machine. You can find the instructions at www.git-scm.com.

### Prerequisite #6: Git repo at https://github.com/redhat-developer-demos/openshift-pipelines-tutorial.git  

You will need the Git repo associated with this workshop.

`git clone https://github.com/redhat-developer-demos/openshift-pipelines-tutorial.git`

After cloning this repo, move into the directory where it is located (typically "openshift-pipelines-tutorial"). This will be the home directory for the remainder of this workshop.

## NOT Prerequisites  

It's worth noting here what you *do not* need to install. You do not need:
* Go language support
* C# (.NET Core) support
* Make, MSBuild or any other build tools

This is because all the build and deploy work is done in our OpenShift cluster. In fact, once the pipeline is built, you could do all your development work using [CodeReady Workspaces](https://developers.redhat.com/products/codeready-workspaces/overview) on, say, a Chromebook.


## Workshop
The following list shows the seven steps that will be used to get our CI/CD pipeline up and running:
1. Create a project
1. Install operator
1. Create Pipeline
1. Create Tasks
1. Create Pipeline Resources
1. Run the Pipeline
1. View the results

<hr>  
 <div style="background-color: cornsilk; margin-left: 20px; margin-right: 20px">
<h4>Operators and Subscriptions Explained</h4>  

Because OpenShift is built on Kubernetes, it supports the concept of "Operators", or pre-built Customer Resource Descriptions (CRD) that include the sometimes many pieces needed to invoke and sustain a solution. In other words, a Kubernetes Operator can be used to start and maintain a complex solution. For example, there is a Kubernetes Operator that allows you to very easily get an instance of MongoDB running in your OpenShift cluster. There are others: Eclipse Che, Elasticsearch, Kafka, and dozens more.  

To invoke an operator *may* involve many steps. You install the Operator and the create a Subscription. In some cases, such as Kafka, you then invoke the API you want. For example, Kafka Connect or Kafka Topic.
  
For OpenShift Pipelines, one quick command will give us all we need.
</div>
<hr>

### Workshop Step 0: Log in
Before any work can begin, you must be logged into your OpenShift cluster with cluster-admin rights. Use the `oc login` command to do this. For example, on my Fedora 32 machine using CRC:

`oc login -u kubeadmin -p 8rynV-SeYLc-h8Ij7-YPYcz https://api.crc.testing:6443`

### Workshop Step 1: Create a project
Create an OpenShift project in which we'll be working.  

`oc new-project pipelines-tutorial`

Example:
![tkn resource ls results](images/oc-new-project.png)

### Workshop Step 2: Install operator
When it comes to installing the OpenShift Pipelines Operator, you have two choices: Use the web UI dashboard or use the command line. For this workshop, we'll be using the command line. 

#### Installing the Pipelines operator using the command line.  
This will install the Pipelines operator and make it available for use.  

`oc apply -f sub.yaml`

Example:
![oc apply -f sub.yaml](images/oc-apply-sub.png)

### Workshop Step 3: Create pipeline  
The next step is to create the pipeline we want to use. The following pipeline is discussed following this image:

![pipeline.png](images/pipeline.png)

#### Line 4: name

This is the name assigned to the pipeline. This name *does not* need to match the name of the file used to create this pipeline. In fact, in this particular example, the file is named "qotd-pipeline.yaml" while the pipeline, *inside of your OpenShift cluster*, is named "qotd-build-and-deploy". It is probably a good idea to make the names match; this is a management issue.

#### Lines 6 through 10: resource

This pipeline uses two resources: A git repo as the input and a Linux container as the output. That is to say, it will clone a git repo and build an image.

Note that the resource _names_ are *not* the same thing as the resource *values*. In our example, the git repo resource name is "qotd-git", but that has no value. Later, when we create this resource, we'll give it a value, i.e. a URL that points to the repo.

#### Lines 13 through 36: tasks

This section declares what is done and in what order.

Line 13 is a name that we've assigned; it could just as well be called 'foo'. The "taskRef", in lines 14 through 16, is where we call out exactly what to do and where to find it. 

The name "buildah" (line 15) is a built-in *ClusterTask* (line 16), a task that is built into the OpenShift Pipelines. You can see this and all the other ClusterTasks by running the command `tkn clustertask ls`. A ClusterTask is available across *all* namespaces in the OpenShift cluster, while tasks that we have created are limited to the namespace in which they are created.

I'm using buildah to build the image. Since I have the file "Dockerfile" in my source code, this is an excellent choice. I can use the Dockerfile while developing and testing on my local machine, and then when I push the source to the git repo and use it in my pipeline, I can be assured the same build steps and resulting image will be used. I'm in control, and I like that.

In line 30 we declare a task that we created (creating a Task happens later in this workshop). Notice line 36, which specifies that this task runs only after the task "image-build" is completed.

With that knowledge, let's create the pipeline:

#### Command to create the pipeline  

`oc create -f qotd-pipeline.yaml`

Example:
![oc apply -f qotd-pipeline.yaml](images/oc-apply-pipeline.png)  


### Workshop Step 4: Create tasks  

After our pipeline builds the image and places it into the OpenShift project's registry, the only task we need to create in our namespace is the "apply-manifests" task. As good fortune would have it, we don't even need to create this from scratch. It's a part of the OpenShift Pipelines catalog, a collection of open source, reusable tasks. The catalog can be found at https://github.com/openshift/pipelines-catalog.

Here is the command to create the "apply-manifests" task:  

`oc create -f apply_manifest_task.yaml`

The "apply-manifests" task will find the YAML files in the git repo's "k8s" directory and apply them to our application. Knowing this, our source repos will have a "k8s" directory with three manifests: a Deployment Config, a Service, and a Route. You can view these by looking in the source repos:

* https://github.com/redhat-developer.demos/qotd.git
* https://github.com/donschenck/qotd-csharp.git


### Step 5: Create pipeline resources  

We need to create two resources:
* qotd-git - defines the location of the github repo containing the source code to be compiled into an image
* qotd-image - defines the location of the created image

We'll need to run the command `tkn resource create` twice, once for each resource. Here are the values needed:

`tkn resource create`  
Name: qotd-git  
Type: git  
Url: https://github.com/redhat-developer-demos/qotd.git  **OR** 
  https://github.com/donschenck/qotd-csharp.git  
Revision: (leave blank)  

**NOTE**: The choice of URL depends on whether you want to use the Go source code or the C# source code. By design, they both yield the same results when they run.  

`tkn resource create`  
Name: qotd-image  
Type: image  
Url: image-registry.openshift-image-registry.svc:5000/pipelines-tutorial/qotd:latest  
Digest: (leave blank)  

We can view the results by running `tkn resource ls`  

![tkn resource ls results](images/tkn-resource-ls.png)

### Workshop Step 6: Run pipeline  
At this point, we have a pipeline, tasks that make up the pipeline, and resources to be used by the build. We can see what we have by running some commands.

`tkn task ls` to see a list of tasks:

Example:
![tkn task ls results](images/tkn-task-ls.png)

`tkn pipleline ls` to see a list of pipelines:

Example:
![tkn pipeline ls results](images/tkn-pipeline-ls.png)

Now what we have the name of the pipeline, we can start it. After it is started, the terminal will return the command you will use to watch the progress of the build.

When you are prompted for the resources to use, use the default values.

`tkn pipeline start qotd-build-and-deploy`

Example:
![tkn pipeline start](images/tkn-pipeline-start.png)

![pipeline log](images/pipeline-log.png)

### Workshop Step 7: View results  

`oc get routes`

Use the route in your browser, or run the cURL command in your terminal, to navigate to the results. Refresh your browser several times to see random results.

Valid routes include:  
"/"  
"/version"  
"/quotes"  
"/quotes/random"  
"/quotes/0"  
"/quotes/1"  
"/quotes/2"  
"/quotes/3"  
"/quotes/4"  
"/quotes/5"  

Example:  
`curl foo/quotes/random`

### Workshop Optional Step 8: Push changes and see results

This section is purposely kept terse. It is assumed that you have the technical skills for perform the steps listed.

Step 1. Fork the repo at https://github.com/redhat-developer-demos/qotd.git to your own Github account.  
Step 2. Clone your repo to your local machine.  
Step 3. Change the source code in "main.go" in the "qotd" repo that you forked and cloned to your local PC.  
Step 4. Commit and push the change to your repo.  
Step 5. Delete the qotd-git resource: `tkn resource delete qotd-git`  
Step 6. Create the resource "qotd-git", pointing it to your repo. `tkn resource create`  
Step 7. Run the pipeline start command again. `tkn pipeline start qotd-build-and-deploy --last`    
Step 8. When the build is complete, view the resulting changes.  

Hint: This is a good command to run in a bash terminal session:  
`while true; do sleep 1; curl <<your route here>>/quotes/random; done`

Or in PowerShell:  
`While($true) { curl <<your route here>>/quotes/random; sleep 1 }`

<p style="text-align: center;">This marks the end of this workshop.</p>