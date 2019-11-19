# tekton-examples
A practice repo to collect examples of using tekton with kubernetes

## Notes on Tekton

Tekton is made up of **6 main components**:

- Step 
-- run commands
- Task 
-- list of steps
- Pipeline 
-- graph of tasks
- Pipeline Resource 
-- a resource that is declared and then referenced and used in tasks and pipelines
- Task Run 
-- invoke a task  
- Pipeline Run
-- invoke a pipeline

#### Useful Youtube Videos:

[https://www.youtube.com/watch?v=V0LpYdnTpsg](https://www.youtube.com/watch?v=V0LpYdnTpsg)

[https://www.youtube.com/watch?time_continue=691&v=Bt-4LOkXJLU&feature=emb_logo](https://www.youtube.com/watch?time_continue=691&v=Bt-4LOkXJLU&feature=emb_logo)


#### References:

[https://github.com/tektoncd/pipeline/blob/master/docs/install.md](https://github.com/tektoncd/pipeline/blob/master/docs/install.md)

[https://github.com/tektoncd/pipeline/blob/master/docs/tutorial.md](https://github.com/tektoncd/pipeline/blob/master/docs/tutorial.md)

[https://github.com/tektoncd/pipeline/tree/master/examples](https://github.com/tektoncd/pipeline/tree/master/examples)

[https://github.com/tektoncd/dashboard](https://github.com/tektoncd/dashboard)

Notes on setting up the dashboard:

Simplest way to access the dashboard locally, use port-forward:

    $ kubectl get pods --namespace tekton-pipelines

Find the dashboard pod:

    tekton-dashboard-xxx-xxx

View the dashboard by using port-forward:

    $ kubectl port-forward tekton-dashboard-xxx-xxx -n tekton-pipelines  9097:9097

You should not be able to view the dashboard at:

    http://localhost:9097



## Examples

### Example 1

A simple github read process

Run a simple example where you will set a public github repo as a Pipeline Resource

    $ kubectl create 