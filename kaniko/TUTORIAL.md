# Kaniko Tutorial

Original tutorial:

https://github.com/GoogleContainerTools/kaniko/blob/master/docs/tutorial.md

I've just taken those steps and made them simpler - I hope - less fumbling, more copy paste and getting examples working with less effort.

This is assuming you have a google cloud environment set up and a kubernetes cluster with google cloud. If you do not I would suggest rather following the link above and adjusting those steps to your needs.

## Tutorial 1

This tutorial is using

    --context=dir://workspace

We are going to build an image using files we place in a folder location (/workspace), we will do this by using a volumeMount and mount the contents of a configmap into the running Kaniko pod.

### Setup

I had an idea to create a dockerfile with a configmap, because why not. It's more conventient for creating POCs (less ssh and files to place in exact locations etc).

And create the docker-registry secret:

    $ kubectl create secret docker-registry regcred --docker-server=index.docker.io --docker-username=username --docker-password=password --docker-email=email@address.com

Create these two files:
- configmap.yaml
- pod.yaml

Commands:

    $ kubectl create -f ./example-1/configmap.yaml
    $ kubectl create -f ./example-1/pod.yaml

Note:

Make sure your secret regcred is correctly setup with the right values.


## Tutorial 2

This tutorial is using

    --context=git://github.com/cariza/ulmaceae

We are going to build an image using files from a githun repo (github.com/cariza/ulmaceae), we will do this by using a volumeMount and mount the contents of a configmap into the running Kaniko pod.


Create this file:

- pod.yaml

Command:

    $ kubectl create -f ./example-2/pod.yaml

Some notes on these:

This argument tells Kaniko where to find the Dockerfile

    --dockerfile=/workspace/dockerfile

This argument refers to how Kaniko can expect to get access to your source code/project. In the two examples above we used these 2 contexts:

    --context=dir://workspace
    --context=git://github.com/youruser/repo

This argument refers to the destination Kaniko will be sending the built and tagged image (provided Kaniko has permission - eg using the docker-registry secret credentials):

    --destination=<user-name>/<repo>