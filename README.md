# tekton-examples

A practice repo to collect examples of using tekton with kubernetes.

Please note, this is just a reference for practicing with, this is not necessarily production ready.

** Information on how to run the examples are below the references **

## Tekton setup notes

These steps were taken and tweaked from this source:

[https://github.com/tektoncd/pipeline/blob/master/docs/install.md](https://github.com/tektoncd/pipeline/blob/master/docs/install.md)

Setup some environment variables:

    export CLUSTER_NAME=tekton-prac
    export CLUSTER_ZONE=europe-west4-a
    export NUM_NODES=2

Check values are set:

    echo $CLUSTER_NAME && echo $CLUSTER_ZONE && echo $NUM_NODES

Create the gc cluster:

    $ gcloud container clusters create $CLUSTER_NAME --zone=$CLUSTER_ZONE --num-nodes=$NUM_NODES

    $ gcloud container clusters list

To delete the cluster (if you want to start from scratch again):

    $ gcloud container clusters delete $CLUSTER_NAME

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

* Step 

    run commands

* Task 

    list of steps

* Pipeline 

    graph of tasks

* Pipeline Resource 

    a resource that is declared and then referenced and used in tasks and pipelines

* Task Run 

    invoke a task  

* Pipeline Run

    invoke a pipeline


#### Useful Youtube Videos:

[https://www.youtube.com/watch?v=V0LpYdnTpsg](https://www.youtube.com/watch?v=V0LpYdnTpsg)

[https://www.youtube.com/watch?time_continue=691&v=Bt-4LOkXJLU&feature=emb_logo](https://www.youtube.com/watch?time_continue=691&v=Bt-4LOkXJLU&feature=emb_logo)


#### References:

[https://github.com/tektoncd/pipeline/blob/master/docs/install.md](https://github.com/tektoncd/pipeline/blob/master/docs/install.md)

[https://github.com/tektoncd/pipeline/blob/master/docs/tutorial.md](https://github.com/tektoncd/pipeline/blob/master/docs/tutorial.md)

[https://github.com/tektoncd/pipeline/tree/master/examples](https://github.com/tektoncd/pipeline/tree/master/examples)

[https://github.com/tektoncd/dashboard](https://github.com/tektoncd/dashboard)

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

    $ kubectl port-forward tekton-dashboard-xxx-xxx -n tekton-pipelines  9097:9097

You should now be able to view the dashboard at:

[http://localhost:9097](http://localhost:9097)




# Examples

### Example 1

A simple github read process

Run a simple example where you will set a public github repo as a Pipeline Resource

Run these yaml scripts in this order:

    $ kubectl create -f ./example-github-read/resource.yaml
    $ kubectl create -f ./example-github-read/task.yaml
    $ kubectl create -f ./example-github-read/pipeline.yaml 
    $ kubectl create -f ./example-github-read/pipeline-run.yaml 

Some gotchas:

- Some of the resources depend on other resources existing before they run
- TODO: Check if there's a way to flag a resource to only run once another resource exists

Check the dashboard:

[http://localhost:9097/#/namespaces/default/pipelineruns/pipelinerun-test](http://localhost:9097/#/namespaces/default/pipelineruns/pipelinerun-test)

