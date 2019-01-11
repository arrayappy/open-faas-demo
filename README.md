# OpenFaaS Demo

> OpenFaaS® (Functions as a Service) is a framework for building Serverless functions with Docker and Kubernetes

Official Resources:
- https://www.openfaas.com/
- https://github.com/openfaas/faas

### Serverless

**Serverless** describes a cloud computing model in which all aspects of server management (provisioning, scaling, maintenance, etc) is managed by the cloud provider, abstracted and hidden from the developer.

**FaaS - Functions as a Service** is a specific category of serverless computing. FaaS platforms support the deployment of discrete functions, units of code which handle a piece of business logic

**Examples (other than OpenFaaS):**
- [AWS Lambda](https://aws.amazon.com/lambda/)
- [Google Cloud Functions](https://cloud.google.com/functions/)
- [Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/)

## Setup

### Deploy  OpenFaaS

Install and start a minikube cluster: https://kubernetes.io/docs/setup/minikube/

Deploy the OpenFaaS platform:
```sh
kubectl apply -f ./faas-netes/namespaces.yml
kubectl apply -f ./faas-netes/k8s/
```
You should now be able to open the OpenFaaS web UI, AKA the "gateway" service:
```sh
minikube service -n openfaas gateway
```

### Install Tools and Configure Environment
Install FaaS CLI
```sh
brew install faas-cli
```

Setup local Docker cli to use minikube (so we don't need to push images to a remote repo):
```sh
eval $(minikube docker-env)
```

 Save the gateway service url in the env variable `OPENFAAS_URL`. The FaaS CLI will automatically use this later for deploying functions.
```sh
export OPENFAAS_URL="http://$(minikube ip):31112"
```

## Create, Build, and Deploy Serverless Functions

### Create a new function

Make a new directory somewhere and use the FaaS CLI to generate the function template:
```sh
mkdir /path/to/my-functions
cd /path/to/my-functions
faas-cli new --lang ruby hello-demo
```

This creates some new files in the functions directory:
```
./hello-demo.yml
./hello-demo/Gemfile
./hello-demo/handler.rb
```

The yaml file configures the faas-cli for building, pushing and deploying the function.
Edit `./hello-demo.yaml` so the image has a versioned tag, not `:latest`. (This allows kubernetes, with image pull policy = IfNotPresent, to use the local image and we don't have to push to a registry).
```yaml
    image: hello-demo:0.1.0
```

The handler, `./hello-demo/handler.rb`, is where all the magic happens. Edit it to do whatever you like:
```rb
require 'time'
class Handler
  def run(req)
    t = Time.now
    return "Hello Demo! It is currently #{t}. Your input is:\n\t#{req}"
  end
end
```

### Build and Deploy
```sh
faas-cli build -f ./hello-demo.yml
faas-cli deploy -f ./hello-demo.yml
```

### Invoke

With curl:
```sh
curl $OPENFAAS_URL/function/hello-demo -d "demo guy"
```
or with the cli:
```sh
echo -n "wasup" | faas-cli invoke hello-demo
```

### Check out some functions built by the community

A QR Code generator - deploy by Docker image
```sh
faas-cli deploy --image=faasandfurious/qrcode --name=qrcode --fprocess="/usr/bin/qrcode"

curl $OPENFAAS_URL/function/qrcode --data "https://www.codecademy.com/" > qrcode.png
```

A youtube video downloader - deploy by stack.yml file:
```sh
faas-cli deploy --gateway $OPENFAAS_URL -f https://raw.githubusercontent.com/faas-and-furious/youtube-dl/master/stack.yml

curl $OPENFAAS_URL/function/youtubedl --data "https://www.youtube.com/watch?v=hn5Hlusj6Nc" > youtube.mov
```

## Cleanup

Remove the functions:
```sh
faas-cli rm hello-demo
faas-cli rm qrcode
faas-cli rm youtubedl
```

Delete the FaaS deployment:
```sh
kubectl delete -f ./faas-netes/k8s/
kubectl delete -f ./faas-netes/namespaces.yml
```

Spin down the kubernetes cluster:
```sh
minikube stop
```

Unset the Docker environment:
```sh
eval $(docker-machine env -u)
```
