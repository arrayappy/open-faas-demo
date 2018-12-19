
# OpenFaaS Demo

[OpenFaaS](https://github.com/openfaas/faas) is a framework for packaging code/binaries/contianers as Serverless Functions for Docker/Kubernetes.


## Setup

### Deploy OpenFaas

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
You should be able to open the gateway web UI, e.g. at http://192.168.99.100:31112

**Save the gateway host and port for later**:
```sh
export faas_gw_url=http://192.168.99.100:31112
```

Setup local docker cli to use minikube
```sh
eval $(minikube docker-env)
```

### Install FaaS CLI

```sh
brew install faas-cli
```

## Create, Build, and Deploy Serverless Functions

Create a new function:
```sh
cd functions
faas-cli new --lang ruby hello-demo
```

Edit `./functions/hello-demo.yaml` with the valid gateway URL, and tag the image with a version, e.g.:
```yaml
provider:
  name: faas
  gateway: http://192.168.99.100:31112 # <- replace with actual url
functions:
  hello-demo:
    lang: ruby
    handler: ./hello-demo
    image: hello-demo:1.0.0
```

Edit `./functions/hello-demo/handler.rb` to do whatever you like:
```rb
require 'time'
class Handler
  def run(req)
    t = Time.now
    return "Hello Codecademy Demo! It is currently #{t}"
  end
end
```

Build:
```sh
faas-cli build -f ./hello-demo.yml
```

Deploy:
```sh
faas-cli deploy -f ./hello-demo.yml
```

Invoke:
```sh
curl $faas_gw_url/function/hello-demo
```

## More

List, explore, add, delete functions in the gateway UI:
```sh
open $faas_gw_url
```
