
# OpenFaaS Demo

[OpenFaaS](https://github.com/openfaas/faas) is a framework for packaging code/binaries/containers as serverless functions for Docker/Kubernetes.

## Setup

### Deploy OpenFaaS

Install and start a minikube cluster: https://kubernetes.io/docs/setup/minikube/

Deploy the OpenFaaS platform (source: https://github.com/openfaas/faas-netes)
```sh
kubectl apply -f ./faas-netes/namespaces.yml
kubectl apply -f ./faas-netes/k8s/
```

List the minikube services:
```sh
minikube service -n openfaas list
```
You should be able to open the OpenFaaS web UI at the `gateway` service endpoint, e.g. at http://192.168.99.100:31112

**Pro tip:** save this url in the env variable `OPENFAAS_URL`. The `faas-cli` will automatically use this later on.
```sh
export OPENFAAS_URL=http://192.168.99.100:31112
```

Setup local docker cli to use minikube (so we don't need to push images to a remote repo):
```sh
eval $(minikube docker-env)
```

### Install FaaS CLI

```sh
brew install faas-cli
```

## Create, Build, and Deploy Serverless Functions

### Create a new function

```sh
cd functions
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
    image: hello-demo:1.0.0
```

`./hello-demo/handler.rb`, is the meat of the function. Edit it to do whatever you like:
```rb
require 'time'
class Handler
  def run(req)
    t = Time.now
    return "Hello #{req}! It is currently #{t}"
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

Spin down the cluster:

```sh
minikube stop
```
