# Kaniko Tutorial

Original tutorial:

https://github.com/GoogleContainerTools/kaniko/blob/master/docs/tutorial.md

I've just taken those steps and made them simpler - I hope - less fumbling, more copy paste and getting examples working with less effort.

This is assuming you have a google cloud environment set up and a kubernetes cluster with google cloud. If you do not I would suggest rather following the link above and adjusting those steps to your needs.

## Tutorial 1

This tutorial is using

    --context=dir://workspace

We are going to build an image using files we place inside the running kaniko pod at a folder location (/workspace), we will do this by using a volumeMount and mount the contents of a configmap into the running Kaniko pod.

### Setup

Let's create a dockerfile with a configmap, because why not. It's more convenient for creating POCs (less ssh and files to place in exact locations etc).

And create the docker-registry secret:

    $ kubectl create secret docker-registry regcred --docker-server=index.docker.io --docker-username=username --docker-password=password --docker-email=email@address.com

Create these two files:
- configmap.yaml
- pod.yaml

Note

You need to just tweak the pod.yaml file and replace 

    args: [
        ...
        "--destination=index.docker.io/youruser/repo:example1"
        ...
    ]

For example my account and repo would be:

    index.docker.io/cariza/tekton-prac:example1

You can change "example1" to be any tag you would like.

You can have a look at my tests here. You can mimic this repo with your own account and repo, an empty repo will work fine. 

    https://hub.docker.com/repository/docker/cariza/tekton-prac

Just make sure you provide the right details to the docker-registry secret mentioned above. You won't have permission to edit my docker hub repo, but you can provide your authentication details and be able to edit your docker hub repo.

Commands:

    $ kubectl create -f ./example-1/configmap.yaml
    $ kubectl create -f ./example-1/pod.yaml

Follow the logs:

    $ kubectl logs kaniko-example-1

Note:

Make sure your secret regcred is correctly setup with the right values.


## Tutorial 2

This tutorial is using

    --context=git://github.com/cariza/empty-hello-image

We are going to build an image using files from a githun repo (you can use mine, or fork it ot create your own - github.com/cariza/empty-hello-image), we will do this by using a volumeMount and mount the contents of a repo into the running Kaniko pod.

This tutorial will also work if you did not do Tutorial 1, this will not use docker hub, but rather focus on a github repo. And because it's public you won't nee authentication details for this tutorial.

Create this file:

- pod.yaml

Command:

    $ kubectl create -f ./example-2/pod.yaml

Follow the logs:

    $ kubectl logs kaniko-example-2

## Kaniko Notes:

### The Arguments we used in these tutorials

- --dockerfile
- --context
- --destination

#### --dockerfile

This argument tells Kaniko where to find the Dockerfile

    --dockerfile=/workspace/dockerfile

This is the file Kaniko will use to build the image. Imagine Kaniko running something like this (just hypothetically):

    $ docker build -t tag -f /workspace/dockerfile

#### --context    

This argument refers to how Kaniko can expect to get access to your source code/project. In the two examples above we used these 2 contexts:

    --context=dir://workspace
    --context=git://github.com/youruser/repo

Official docs show more options: [https://github.com/GoogleContainerTools/kaniko#kaniko-build-contexts](https://github.com/GoogleContainerTools/kaniko#kaniko-build-contexts)

#### --destination

This argument refers to the destination Kaniko will be sending the built and tagged image (provided Kaniko has permission - eg using the docker-registry secret credentials):

    --destination=<user-name>/<repo>


## Run kaniko locally using docker

If you want to tinker with Kaniko even more, and investigate it locally (not with kubernetes - with docker only). This can be a nice way to isolate problems and test locally without creating a complex environment.

Docker pull:

    $ docker pull gcr.io/kaniko-project/executor:latest

To run you would need to create a local dockerfile and use a volume to pass it in to kaniko:

 Dockerfile

Run the imate with docker (from the same folder as your Dockerfile):

    $ docker run --rm --name test-kaniko --volume $(pwd)/Dockerfile:/workplace/Dockerfile gcr.io/kaniko-project/executor:latest --no-push --dockerfile /workplace/Dockerfile

