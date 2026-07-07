# Capstone-Deploy-WordPress-MySQL-on-Kubernetes

A hands-on Kubernetes capstone project that deploys a WordPress application with MySQL using Kubernetes resources such as Namespace, Secret, ConfigMap, StatefulSet, Persistent Volume Claim, Services, Deployment, Probes, and Horizontal Pod Autoscaler.

## Architecture

```text
Browser
   |
   | kubectl port-forward localhost:8080
   v
WordPress NodePort Service
   |
   v
WordPress Deployment
   |
   |-- WordPress Pod 1
   |-- WordPress Pod 2
   |
   v
MySQL Headless Service
   |
   v
MySQL StatefulSet
   |
   v
MySQL Pod: mysql-statefulset-0
   |
   v
Persistent Volume Claim
   |
   v
Persistent Storage

## Technologies Used

Kubernetes
kind
Docker Desktop
kubectl
WordPress
MySQL 8.0
Metrics Server
Horizontal Pod Autoscaler

## Kubernetes Concepts Used

Namespace
Secret
ConfigMap
Persistent Volume Claim
StatefulSet
Headless Service
Deployment
NodePort Service
Resource Requests and Limits
Liveness Probe
Readiness Probe
Horizontal Pod Autoscaler

## Project Resources
 
Resource	         Name	                       Purpose

Namespace	       capstone	              Separates all project resources
Secret	           mysql-secret	          Stores MySQL credentials
ConfigMap	       wordpress-config	      Stores MySQL host and database name
StatefulSet	       mysql-statefulset	  Runs MySQL with stable identity and storage
Headless Service	mysql-headless	      Provides stable DNS for MySQL
Deployment	        wordpress	          Runs two WordPress replicas
NodePort Service	wordpress	          Exposes WordPress
HPA	                wordpress-hpa	      Scales WordPress Pods based on CPU
PVC	               MySQL volume claim	  Keeps MySQL data after Pod recreation

## Prerequisites

## Install:

Docker Desktop
kind
kubectl

## Check installation:

docker --version
kind --version
kubectl version --client
Create the Kubernetes Cluster
kind create cluster --name capstone-cluster
kubectl get nodes

## Expected node:

capstone-cluster-control-plane   Ready

## Create Namespace

kubectl create namespace capstone
kubectl config set-context --current --namespace=capstone

## Verify:

kubectl config view --minify

## Create MySQL Secret

Create a local file named .env.mysql:

MYSQL_ROOT_PASSWORD=your-root-password
MYSQL_DATABASE=wordpress
MYSQL_USER=wordpressuser
MYSQL_PASSWORD=your-wordpress-password

Do not commit this file to GitHub.

## Create the Kubernetes Secret:

kubectl create secret generic mysql-secret --from-env-file=.env.mysql -n capstone

## Verify:

kubectl get secret mysql-secret -n capstone

## Deploy MySQL

Apply the MySQL Headless Service and StatefulSet:

kubectl apply -f mysql.yaml
kubectl get pods -n capstone -w

- Expected Pod:

mysql-statefulset-0   1/1   Running

- Verify persistent storage:

kubectl get pvc -n capstone

- Verify the WordPress database:

kubectl exec -it mysql-statefulset-0 -n capstone -- mysql -u wordpressuser -pYOUR_PASSWORD -e "SHOW DATABASES;"

- Expected database:

wordpress

## Deploy WordPress

- Apply the WordPress ConfigMap and Deployment:

kubectl apply -f wordpress.yaml
kubectl get pods -n capstone -w

- Verify both replicas:

kubectl get deployment wordpress -n capstone

- Expected result:

wordpress   2/2   2   2

## Expose WordPress

Apply the NodePort Service:

kubectl apply -f wordpress-service.yaml
kubectl get svc wordpress -n capstone

For kind, access WordPress using port-forward:

kubectl port-forward svc/wordpress 8080:80 -n capstone

Open in browser:

http://localhost:8080

Complete the WordPress setup wizard and publish a blog post.

## Self-Healing Test

Delete one WordPress Pod:

kubectl get pods -n capstone
kubectl delete pod <wordpress-pod-name> -n capstone
kubectl get pods -n capstone -w

The Deployment automatically creates a replacement Pod.

Delete the MySQL Pod:

kubectl delete pod mysql-statefulset-0 -n capstone
kubectl get pods -n capstone -w

The StatefulSet recreates the Pod with the same name and reconnects the same persistent storage.

Verify MySQL again:

kubectl exec -it mysql-statefulset-0 -n capstone -- mysql -u wordpressuser -pYOUR_PASSWORD -e "SHOW DATABASES;"

The wordpress database and published blog post remain available.

## Horizontal Pod Autoscaler

Apply the HPA:

kubectl apply -f wordpress-hpa.yaml
kubectl get hpa -n capstone

The HPA configuration:

CPU target: 50%
Minimum replicas: 2
Maximum replicas: 10

## Metrics Server

Metrics Server is required for HPA CPU metrics.

Check metrics:

kubectl top nodes
kubectl top pods -n capstone

## Final Verification

kubectl get all -n capstone
kubectl get pvc -n capstone
kubectl get hpa -n capstone

## Stop and Start the Local Cluster

Stop the kind cluster when not practicing:

docker stop capstone-cluster-control-plane

Start it again:

docker start capstone-cluster-control-plane
kubectl get nodes
kubectl get pods -n capstone

## Cleanup

Delete all project resources:

kubectl delete namespace capstone
kubectl config set-context --current --namespace=default

## Key Learnings

Used a StatefulSet for MySQL because databases need stable identity and persistent storage.
Used a Headless Service so MySQL has stable DNS.
Used a Deployment for WordPress because WordPress Pods are interchangeable.
Used ConfigMap for non-sensitive configuration.
Used Secret for database credentials.
Used PVC so MySQL data survives Pod deletion.
Used liveness and readiness probes for WordPress health checks.
Used HPA to scale WordPress Pods based on CPU usage.
Tested Kubernetes self-healing by deleting Pods and observing automatic recreation.