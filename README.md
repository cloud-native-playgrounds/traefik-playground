# traefik-playground

Demonstrating how to setup Traefik on a k3s cluster using arkade and k3d. 

> This is just a learning playground

##  Install k3d
k3d is a little helper to run k3s in docker, where k3s is the lightweight Kubernetes distribution by Rancher. It actually removes millions of lines of code from k8s. If you just need a learning playground, k3s is definitely your choice.

Check out [k3d Github Page](https://github.com/rancher/k3d#get) to see the installation guide.

> When creating a cluster, ``k3d`` utilises ``kubectl`` and ``kubectl`` is not part of ``k3d``. If you don't have ``kubectl``, please install and set up [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/). 

Once you've installed ``k3d`` and ``kubectl``, run
```
k3d create -n traefik-playground
```

```
INFO[0000] Created cluster network with ID 9f3c0c8caf0712b7f59970afd3c5db8159362b6ea34c84788f57b8e41a9011b3
INFO[0000] Created docker volume  k3d-traefik-playground-images
INFO[0000] Creating cluster [traefik-playground]
INFO[0000] Creating server using docker.io/rancher/k3s:v1.0.1...
INFO[0004] SUCCESS: created cluster [traefik-playground]
INFO[0004] You can now use the cluster with:

export KUBECONFIG="$(k3d get-kubeconfig --name='traefik-playground')"
kubectl cluster-info
```

Run ``k3d list`` to make sure the status is running.
```
+--------------------+------------------------------+---------+---------+
|        NAME        |            IMAGE             | STATUS  | WORKERS |
+--------------------+------------------------------+---------+---------+
| traefik-playground | docker.io/rancher/k3s:v1.0.1 | running |   0/0   |
+--------------------+------------------------------+---------+---------+
```

We need to make ``kubectl`` to use the kubeconfig for that cluster.
```
export KUBECONFIG="$(k3d get-kubeconfig --name='traefik-playground')"
```

## Install arkade 
Moving on to [arkade](https://github.com/alexellis/arkade), it provides a simple Golang CLI with strongly-typed flags to install charts and apps to your cluster in one command. Originally, the codebase is derived from [k3sup](https://github.com/alexellis/k3sup) which I've contributed last month. 

```
curl -sLS https://dl.get-arkade.dev | sudo sh
```

Once you've installed it, you should see the following
```
New version of arkade installed to /usr/local/bin
            _             _
  __ _ _ __| | ____ _  __| | ___
 / _` | '__| |/ / _` |/ _` |/ _ \
| (_| | |  |   < (_| | (_| |  __/
 \__,_|_|  |_|\_\__,_|\__,_|\___|

Get Kubernetes apps the easy way

Version: 0.2.2
Git Commit: 9063b6eb16deae5978805f71b0e749828c815490
```

``arkade`` currently does not support non-existing namespace for ``traefik``. Create the namespace first.
```
kubectl create namespace traefik-playground
```

Install Traefik with dashboard via ``arkade``. You can use an alias ``ark`` or ``arkade``. 
```
arkade install traefik2 --dashboard -n traefik-playground 
```

```
Using kubeconfig: /Users/wingkwong/.config/k3d/traefik-playground/kubeconfig.yaml
Client: "Darwin"
"traefik" has been added to your repositories
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "traefik" chart repository
...Successfully got an update from the "openfaas" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
VALUES values.yaml
Command: /Users/wingkwong/.arkade/bin/helm3/helm [upgrade --install traefik traefik/traefik --namespace traefik-playground --values /var/folders/v3/24stf97d7nv35j_9vwppqzy00000gp/T/charts/traefik/values.yaml --set service.type=LoadBalancer --set additional.checkNewVersion=false --set additional.sendAnonymousUsage=false --set dashboard.ingressRoute=true --set additionalArguments={--providers.kubernetesingress}]
Release "traefik" does not exist. Installing it now.
NAME: traefik
LAST DEPLOYED: Mon Apr 20 17:41:21 2020
NAMESPACE: traefik-playground
STATUS: deployed
REVISION: 1
TEST SUITE: None
=======================================================================
=                  traefik2 has been installed                        =
=======================================================================
 Thanks for using arkade!# Get started at: https://docs.traefik.io/v2.0/

# Install with an optional dashboard

arkade install traefik2 --dashboard

# Find your LoadBalancer IP:

kubectl get svc -n kube-system traefik
```

Check out the traefik service 
```
kubectl get svc -n traefik-playground traefik
```

Forwarding from [::1]:8082 -> 8080
```
kubectl port-forward svc/traefik 8082:80 -n traefik-playground &
```

> Currently there are some issues with the dashboard. Reported to ``arkade`` team and pending for a fix.

## Clean up
```
k3d delete -n traefik-playground
```