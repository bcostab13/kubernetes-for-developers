# Kubernetes for Developers

This project have some examples to configure and create objects on a basic Kubernetes cluster. Before start with it, please read the Prerequisites section.

## Prerequisites

Make sure that you have all of these tools installed in your computer:
- Node.js and npm
- Docker (and a Docker hub account or some provider registry)
- Kubernetes
- Minikube or a provider Kubernetes cluster

## Step 1: Create Image

1. Clone the repository. It contains a basic GET service built in node.js and express.

2. For this exercise, we will create 3 namespaces in our cluster, simulating a real environment. There will be: flights, companies and cargo.

```
kubectl create namespace flights
kubectl create namespace companies
kubectl create namespace cargo
```

3. Now, let's create the base image for our fictional scenario. Build the node.js app:

```
npm install
```
Execute the next command to build our docker image:

```
docker build . -t <your-docker-username>/node-app
```

4. Creation of our base image is done. Let's list our local images to validate the last step:

```
docker images
```
You'll get an output similar to:

```
REPOSITORY                    TAG       IMAGE ID       CREATED        SIZE
your-docker/node-app          latest    3216febb3434   3 hours ago    862MB
gcr.io/k8s-minikube/kicbase   v0.0.33   f7ba2bce4549   5 weeks ago    1.06GB
docker/getting-started        latest    157095baba98   4 months ago   27.4MB
```

5. In order for the image to be available, it needs to be uploaded to a repository, like Docker Hub. You can do it with this command:

```
docker push <your-docker-username>/node-app
```

## Step 2: Work with Kubernetes Deployment

1. You can find 3 yaml files. First file contains a simple _deployment_ configuration. Let's create our first deployment using the command:

```
kubectl apply -f 1-Single-Deployment.yml -n flights
```

2. Deployment _spec_ will create 2 replicas of our container (2 replicas of containers per deployment). You can list active pods with:

```
kubectl get pods -n flights
```
**Output:**
```
NAME                      READY   STATUS    RESTARTS   AGE
flights-5c4c85459-hjc9k   1/1     Running   0          38m
flights-5c4c85459-snkf8   1/1     Running   0          38m
planes-56756848db-6wtd8   1/1     Running   0          118m
planes-56756848db-k6db9   1/1     Running   0          118m
```

3. You can also play scaling the number of desired replicas of each deployment:

```
kubectl scale deploy flights --replicas=1 -n flights
```
Then you can list pods to view the effect of the scaling:
```
kubectl get pods -n flights
```
**Output:**
```
NAME                      READY   STATUS    RESTARTS   AGE
flights-5c4c85459-snkf8   1/1     Running   0          38m
planes-56756848db-6wtd8   1/1     Running   0          118m
planes-56756848db-k6db9   1/1     Running   0          118m
```

## Step 3: Adding Networking Layer

1. Even though our apps are deployed, we need to define a way for them to connect without relying on a variant IP. We'll deploy aour first service with kubectl apply:

```
kubectl apply -f 2-Adding-Service.yml -n flights
```

2. List the service.

```
kubectl get svc -n flights
```

3. Now, let's try to connect to flights app from planes pod. First, init bash command in one of the planes pods:

```
kubectl exec -it <planes-pod-id> -n flights -- bash
```
Then, execute curl command:

```
curl flights.flights.svc.cluster.local:8080
```
**Output:**
```
Hello World!
```

## Step 4: Using ConfigMap

1. Finally, create a config map with the command:

```
kubectl apply -f 3-Adding-Configmap.yml -n flights
```

2.You can see the detail of the configmap you've just created, executing:

```
kubectl describe cm flights-cm -n flights
```

3. Enter into one of your flights pods:

```
kubectl exec -it <flights-pod-id> -n flights -- bash
```

Then, execute:
```
echo $MAX_ITEMS
```

You can get the value from de ConfigMap.
