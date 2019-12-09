# tekton-examples


# Examples

These examples are assuming you have made a clone of this repo, or at least copied the files and structured them like they are in this repo.

## Index

- [Example 1](#example-1) Using Tekton Pipelines with a github Repo to run a cat step
- [Example 2](#example-2) A simple docker image building pipeline using Tekton and Kaniko
  - [Side tutorial - Kaniko Tutorial](./kaniko/TUTORIAL.md)

---

# Introduction 

A practice repo to collect examples of using tekton with kubernetes.

Please note, this is just a reference for practicing with, this is not necessarily production ready.

### Just a quick opinion upfront on Tekton and it's usage (opinion is my own)

Upfront my recommendation is not to use Tekton for just a single project that needs CI/CD. There are simpler solutions for a quick solve for CI/CD. 

* [Gitlab CI/CD](https://docs.gitlab.com/ee/ci/)
* AgroCD

Tekton seems more appropriate for building generic pipelines that would solve organisation wide CI/CD requirements. Tekton would be a long term strategy to standardize CI/CD across an organisation, and could create and add more complexity (rather than improve on) if not pre-planned and uniformly adopted.

If you would like to use tools that are built on top of Tekton that abstract away some of the complexity of Tekton:

* [JenkinsX](https://jenkins-x.io/)
* As I find more I will add to this list...

# Setup

[![asciicast](https://asciinema.org/a/286482.svg)](https://asciinema.org/a/286482)

## Tekton setup notes

These steps were taken and tweaked from this source:

[https://github.com/tektoncd/pipeline/blob/master/docs/install.md](https://github.com/tektoncd/pipeline/blob/master/docs/install.md)


### Environment variables

Setup some environment variables:

    export CLUSTER_NAME=tekton-prac
    export CLUSTER_ZONE=europe-west4-a
    export NUM_NODES=2

Check values are set:

    echo $CLUSTER_NAME && echo $CLUSTER_ZONE && echo $NUM_NODES

### Kubernetes cluster

Create the gc kubernetes cluster:

    $ gcloud container clusters create $CLUSTER_NAME \
      --zone=$CLUSTER_ZONE \
      --num-nodes=$NUM_NODES

    $ gcloud container clusters list

**Tip:** To delete the cluster (if you want to start from scratch again):

    $ gcloud container clusters delete $CLUSTER_NAME --zone=$CLUSTER_ZONE

Grant cluster-admin permissions to the current user:

    $ kubectl create clusterrolebinding cluster-admin-binding \
      --clusterrole=cluster-admin \
      --user=$(gcloud config get-value core/account)

Install Tekton Pipelines:

    $ kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

Check installed:

    $ kubectl get pods --namespace tekton-pipelines

You should see some pods in the "tekton-pipelines" namespace:

    tekton-pipelines-controller-...
    tekton-pipelines-webhook-...

## Notes on Tekton

Tekton is made up of **6 main components**:

* **Step**

    run commands

* **Task**

    list of steps

* **Pipeline**

    graph of tasks

* **Pipeline Resource**

    a resource that is declared and then referenced and used in tasks and pipelines

* **Task Run**

    invoke a task (Pipeline runs create Task Runs when run)

* **Pipeline Run**

    invoke a pipeline


#### Useful Youtube Videos:

[https://www.youtube.com/watch?v=V0LpYdnTpsg](https://www.youtube.com/watch?v=V0LpYdnTpsg)

[https://www.youtube.com/watch?time_continue=691&v=Bt-4LOkXJLU&feature=emb_logo](https://www.youtube.com/watch?time_continue=691&v=Bt-4LOkXJLU&feature=emb_logo)


#### References:

[https://github.com/tektoncd/pipeline/blob/master/docs/install.md](https://github.com/tektoncd/pipeline/blob/master/docs/install.md)

[https://github.com/tektoncd/pipeline/blob/master/docs/tutorial.md](https://github.com/tektoncd/pipeline/blob/master/docs/tutorial.md)

[https://github.com/tektoncd/pipeline/tree/master/examples](https://github.com/tektoncd/pipeline/tree/master/examples)

[https://github.com/tektoncd/dashboard](https://github.com/tektoncd/dashboard)

# Dashboard

[![asciicast](https://asciinema.org/a/VzHDh5EeIDeVHQv44X11ORhbL.svg)](https://asciinema.org/a/VzHDh5EeIDeVHQv44X11ORhbL)

### Setup the dashboard for visibility 

Notes on setting up the dashboard:

Install the dashboard with this command:

    $ kubectl apply --filename https://github.com/tektoncd/dashboard/releases/download/v0.2.1/dashboard-latest-release.yaml

Note - Command taken from this source: [https://github.com/tektoncd/dashboard](https://github.com/tektoncd/dashboard)

Simplest way to access the dashboard locally, use port-forward

First check the dashboard pod is running:

    $ kubectl get pods --namespace tekton-pipelines

Find the dashboard pod:

    tekton-dashboard-xxx-xxx

View the dashboard by using port-forward:

    $ kubectl port-forward tekton-dashboard-xxx-xxx -n tekton-pipelines 9097:9097

You should now be able to view the dashboard at:

[http://localhost:9097](http://localhost:9097)




# Examples

These examples are assuming you have made a clone of this repo, or at least copied the files and structured them like they are in this repo.

## Index

- [Example 1](#example-1) Using Tekton Pipelines with a github Repo to run a cat step
- [Example 2](#example-2) A simple docker image building pipeline using Tekton and Kaniko
  - [Side tutorial - Kaniko Tutorial](./kaniko/TUTORIAL.md)

# Example 1

Using Tekton Pipelines with a github Repo to run a cat step

[![asciicast](https://asciinema.org/a/tPHU34Vrqxf3yxpy3yNIVIF5L.svg)](https://asciinema.org/a/tPHU34Vrqxf3yxpy3yNIVIF5L)

### Goal: 

Run a simple example where you will set a public github repo as a **Pipeline Resource**. 

Run these yaml scripts in this order:

    $ kubectl apply -f ./example-1-github-read/

    $ kubectl get pipeline,pipelineresource,task

You should see these resources

    NAME                                AGE
    pipeline.tekton.dev/pipeline-test   1m

    NAME                                      AGE
    pipelineresource.tekton.dev/github-repo   1m

    NAME                        AGE
    task.tekton.dev/read-task   1m

Check the dashboard:

[http://localhost:9097/#/namespaces/default/pipelines](http://localhost:9097/#/namespaces/default/pipelines)

### PipelineRuns

The PipelineRun is the trigger of the pipeline, keep the trigger seperate from the configuration/setup and run it when you want the pipeline to run. Every time we want to trigger the pipeline we would need to have a unique named PipelineRun.

Here are some ideas on how to tackle the pipeline run:

You can manually use the dashboard to trigger the PipelineRuns by going to the "pipelineruns" tab and clicking "Create PipelineRun".

Or we can generate the PipelineRun yaml file using a name + timestamp approach.

### PipelineRun Trigger

```yaml
cat <<EOF >./pipeline-run-$(date +%s).yaml
apiVersion: tekton.dev/v1alpha1
kind: PipelineRun
metadata:
  name: pipelinerun-test-$(date +%s)
spec:
  pipelineRef:
    name: pipeline-test
  resources:
    - name: github-repo
      resourceRef:
        name: github-repo
EOF
```

You would then need to create that PipelineRun by running the yaml file with a "kubecl create -f ..." command:

    $ kubecl create -f pipeline-run-xxxxx.yaml

An approach I felt worked better:

You can create PipelineRuns using the "kubectl create" command directly via **stdin**, this will just create the kubernetes resource and run it but not save a file:

```yaml
cat <<EOF | kubectl create -f -
apiVersion: tekton.dev/v1alpha1
kind: PipelineRun
metadata:
  name: pipelinerun-test-$(date +%s)
spec:
  pipelineRef:
    name: pipeline-test
  resources:
    - name: github-repo
      resourceRef:
        name: github-repo
EOF
```

Response Example:

    pipelinerun.tekton.dev/pipelinerun-test-1574241433 created

Going forwards in these examples I'm going to stick with the approach above, using the stdin approach and not creating yaml files, rather create PipelineRuns directly using the yaml provided as the EOF. I've just shared both approaches so you can tweak the examples for your needs.

Check the dashboard:

[http://localhost:9097/#/namespaces/default/pipelineruns](http://localhost:9097/#/namespaces/default/pipelineruns)

### Review Example 1

Now that you have run through an example, let's quickly visualise the Tekton flow we created in Example 1:

![Tekton Example 1 Diagram](./images/TektonExample1.png)

Some notes:

The PipelineResource is very explicitly added to every resource in the flow. 
Task references the PipelineResource
Pipeline references the PipelineResource
PipelineRun references the PipelineResource

### Use a different git repo

To show the flexibility Tekton offers, let's create a new resource:

```yaml
cat <<EOF | kubectl create -f -
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: another-github-repo
spec:
  type: git
  params:
    - name: url
      value: https://github.com/CariZa/empty-hello-image
EOF
```

Then create a new PipelineRun passing in the new git Resource to the Pipeline


```yaml
cat <<EOF | kubectl create -f -
apiVersion: tekton.dev/v1alpha1
kind: PipelineRun
metadata:
  name: pipelinerun-test-$(date +%s)
spec:
  pipelineRef:
    name: pipeline-test
  resources:
    - name: github-repo
      resourceRef:
        name: another-github-repo
EOF
```

You were able to use the same pipeline for 2 different repos

---

# Example 2

A simple docker image building pipeline using Tekton and Kaniko

### Goal

Run a pipeline that will use a public github repo, build an image using the Dockerfile and run a smoketest to check the image runs as expected.

#### Side Tutorial

Before we try create an image building solution using tekton, let's understand Kaniko in isolation. Then use Kaniko with Tekton.

##### Kaniko

[https://github.com/GoogleContainerTools/kaniko](https://github.com/GoogleContainerTools/kaniko)

##### Kaniko Tutorial

Here is a quick tutorial on building images with Kaniko outside of Tekton (to get an understanding before trying to wrap Tekton around it).

[./kaniko/TUTORIAL.md](kaniko/TUTORIAL.md)

Back to example 2.

### Goal

Run a pipeline that will use a public github repo, build an image using the Kaniko and a specified Dockerfile and run the built image as a test.

Create a Dockerhub Secret

In order for Kaniko to connect with dockerhub you need to create the kubernetes secret:

    $ kubectl create secret docker-registry regcred \
    --docker-server=index.docker.io \
    --docker-username=username \
    --docker-password=password \
    --docker-email=email@address.com

Run the scripts:

    $ kubectl apply -f ./example-2-build-image/

#### PipelineRun Trigger

```yaml
cat <<EOF | kubectl create -f -
apiVersion: tekton.dev/v1alpha1
kind: PipelineRun
metadata:
  name: build-pipeline-test-$(date +%s)
spec:
  pipelineRef:
    name: build-pipeline
  resources:
  - name: github-repo
    resourceRef:
      name: github-repo
  - name: dockerhub-image
    resourceRef:
      name: dockerhub-image
  podTemplate:
    volumes:
    - name: kaniko-secret
      secret:
        secretName: regcred
        items:
        - key: .dockerconfigjson
          path: .docker/config.json
EOF
```

In order for the .docker/config.json to exist you need to create a configmap to that you can use as a volume mount:

This will be added to the step config of the Task:

```yaml
  steps:
  ...
  - name: build
    ...
    volumeMounts:
    - name: kaniko-secret
      mountPath: /builder/home
```

The "kaniko-secret" is registered in the PipelineRun, it's important to keep that in mind.

Once your PipelineRun has completed you can go to your docker hub repo and check that a new image has been created and pushed to docker hub with the tag you provided.

