# How to run

Start minikube
```bash
minikube start
eval $(minikube docker-env)   # This allows k8 to use locally built images
```

Build the flask app Docker image
```bash
docker build -t cloudhack_mongodb -f flask-app-image.dockerfile .
```

Apply all the `.yaml` files OR Run the start script
```bash
kubectl apply -f secret.yaml
kubectl apply -f deployments.yaml
kubectl apply -f services.yaml
kubectl get all -n mongodb-namespace
# OR
./start.sh
```

View all the services, deployments and pods running
```bash
kubectl get all -n mongodb-namespace
```

To view logs of the flask app run the following
```bash
kubectl logs --follow service/flask-app-service -n mongodb-namespace
```

## Exposing the Flask App and Admin Console

This has been done using port forwarding. Fun the following, each in its own terminal
```bash
kubectl port-forward deployment.apps/flask-app-deployment 28016:5001 -n mongodb-namespace


kubectl port-forward replicaset.apps/mongodb-express-deployment-58b4bfcffd 28015:8081 -n mongodb-namespace
```

The flask app will be available at http://localhost:28016/ and the `mongodb-express` Admin console is available at http://localhost:28015/

## Screenshots

One post and the corresponding `mongodb` entry are shown.
<p align = "center">
    <img src = "imgs/flask-app.png"/>
</p>

<p align = "center">
    <img src = "imgs/admin3.png"/>
</p>

## Stop the server

Either manually 
```bash
kubectl delete service/mongodb-express-service -n mongodb-namespace
kubectl delete service/mongodb-service -n mongodb-namespace
kubectl delete deployment.apps/mongodb-deployment -n mongodb-namespace
kubectl delete deployment.apps/mongodb-express-deployment -n mongodb-namespace
kubectl delete deployment.apps/flask-app-deployment -n mongodb-namespace
kubectl delete service/flask-app-service -n mongodb-namespace
# OR
./stop.sh
```


## Pre-Requisites/ Pre-Installation:
1. Docker ([Windows](https://docs.docker.com/desktop/windows/install/) | [Ubuntu](https://docs.docker.com/engine/install/ubuntu/#:~:text=Install%20from%20a%20package&text=Go%20to%20https%3A%2F%2Fdownload,version%20you%20want%20to%20install) | [MacOS](https://docs.docker.com/desktop/mac/install/))
2. Kubernetes ([Windows](https://birthday.play-with-docker.com/kubernetes-docker-desktop/) | [Ubuntu](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) | [MacOS](https://birthday.play-with-docker.com/kubernetes-docker-desktop/))

## File Structure
```
.
|-- README.md
|-- app
|   |-- app.py
|   |-- requirements.txt
|   `-- templates
|       |-- base.html
|       |-- create-post.html
|       |-- edit-post.html
|       `-- home.html
|-- configmap.yaml
|-- deployments.yaml
|-- flask-app-image.dockerfile
|-- secret.yaml
`-- services.yaml
```
The `app` directory contains all the code pertaining to the flask app. You are only required to configure the mongo connection string variables as specified in `app.py`.  
The `flask-app-image.dockerfile` should specify the insructions to assemble the docker image for the flask app.  
The `.yaml` files in the root directory are to specify the kubernetes manifests that will bring up your microservices deployment of the problem statement.

## Task Breakdown/ Deliverables:

1. MongoDB Server
    1. Use the `mongo` image publicly available on DockerHub. Read through the configuration and other details of the image [here](https://hub.docker.com/_/mongo). Note down the necessary environment variables to be configured.
    2. Create the `Deployment` for the mongodb server under `deployments.yaml`. Remember to configure the ports and setup the environment variables correctly.

    3. Environment variables such as username, password, etc. are sensitive information and are defined as a `Secret`. Define a `secret.yaml` file to hold the sensitive information required by the mongodb server. You may create a secret _using a configuration file_ and use the secret in your deployment as _an environment variable_.  [Read more](https://newrelic.com/blog/how-to-relic/how-to-use-kubernetes-secrets).
    4. Create a `Service` for the mongodb server under `services.yaml`.

2. Mongo-Express Web Service
    1. Use the `mongo-express` image. Note down the necessary environment variables like before from [here](https://hub.docker.com/_/mongo-express).
    2. Define a `configMap` to store the mongodb server url. As above, use the configmap to configure the container with environment variables. [Read more](https://kubernetes.io/docs/concepts/configuration/configmap/).
    3. Create a `Deployment` for the mongo-express service under `deployments.yaml` and configure the necessary ports and environment variables (drawn from the secret and configmap).
    4. Also define a `service` for the pod under `services.yaml`.

3.  Flask WebApp
    1. Use the image created from the `flask-app-image.dockerfile`.
    2. Create a `Deployment` for the flask app under `deployments.yaml`.
    3. Also define a `Service` for the pod in `services.yaml`.  

## Bringing it all together
Bring up all the microservices.
Once all the microservices are up and running,
1. Inside the flask-app pod, write and run a python script to insert records into the mongodb database. Insert into: database = 'blog' and collection = 'posts'.
2. Run `app.py` inside the pod. Visit `http://localhost:<port>/` to view the Blog App. The Home Page should display the records inserted into the database in the previous step.
<p align = "center">
    <img src = "https://user-images.githubusercontent.com/56164920/158070358-d37498a4-1712-4048-bf19-3dfc86a214ef.png"/>
</p>

3. You can view the database frontend exposed by mongo-express. To do so, on your browser, navigate to`EXTERNAL_IP:port` exposed by the mongo-express service. Here is the sample output:  

<p align = "center">
    <img src = "https://user-images.githubusercontent.com/56164920/158070411-3dff479d-ee7f-4eeb-b38f-92ccc221c6aa.png"/>
</p>

## MongoDB CLI
You can also play around with the Mongo CLI. Refer [Mongo Shell Guide](https://docs.mongodb.com/manual/reference/mongo-shell/)

Login to the mongo shell and -

1. Create a Database
2. Insert a Collection
3. Fill it with records

Feel free to play around for brownie points!

## All Done! :)

## Extras

https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/

Start all deployments

```bash
minikube start

./start.sh

kubectl port-forward replicaset.apps/mongodb-express-deployment-58b4bfcffd 28015:8081 -n mongodb-namespace

kubectl port-forward deployment.apps/flask-app-deployment 28016:5001 -n mongodb-namespace

kubectl logs --follow service/flask-app-service -n mongodb-namespace
```

Docker container

```bash
docker build -t cloudhack_mongodb -f flask-app-image.dockerfile .

docker run -p 5001:5001 27017:27017 cloudhack_mongodb
```

Stop all

```bash
./stop.sh
```

Experimental

```bash
eval $(minikube docker-env)

kubectl create -f deployments.yaml

kubectl delete -f secret.yaml
kubectl delete -f deployments.yaml
kubectl delete -f services.yaml

kubectl delete deployment.apps/mongo-deployment

docker run -it --rm \
    --name mongo-express \
    -p 8081:8081 \
    -e ME_CONFIG_MONGODB_AUTH_DATABASE="blog" \
    -e ME_SERVER_ROOT_URL="http://0.0.0.0" \
    -e ME_CONFIG_MONGODB_SERVER="mongo" \
    -e ME_CONFIG_BASICAUTH_USERNAME="mongousername" \
    -e ME_CONFIG_BASICAUTH_PASSWORD="mongopassword" \
    mongo-express

mongosh mongodb://mongousername:mongopassword@mongo:28015

mongosh -u mongousername -p mongopassword --host localhost --port 28015

mongosh -u mongousername -p mongopassword --host mongo --port 28015
```