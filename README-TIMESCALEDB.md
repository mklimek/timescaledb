# Run TimescaleDB in Minikube

## Build with docker compose

`docker compose up -d --build`

`docker tag node-kubernetes-web node-kubernetes-web:v1`

Run the migrations and seed the database:

`docker-compose exec web knex migrate:latest`
`docker-compose exec web knex seed:run`


Test it out at:

- [http://localhost:3000](http://localhost:3000)
- [http://localhost:3000/todos](http://localhost:3000/todos)

### All K8s resources will be created under "node" namespace

## Add image to minicube to make K8s be able to fetch local image

`minikube image load node-kubernetes-web:v1`

## Create a secret object with credentials to TimescaleDB

`kubectl apply -f ./kubernetes/secret.yaml`

## Create the volume

`kubectl apply -f ./kubernetes/volume-local.yaml`

## Create the volume claim

`kubectl apply -f ./kubernetes/volume-claim.yaml`

## Mount your local directory to minicube 

Postgres will use minikube location as hostPath for Persistent Volume.

In root of the project:

`mkdir -p datadir/postgres`

`minikube mount datadir/postgres:/data/postgresdata2`

## Postgres

Create deployment:

`kubectl create -f ./kubernetes/postgres-deployment.yaml`

if it fails because "could not access directory "/var/lib/postgresql/data": Permission denied"

then restart minikube:

`minikube stop`

`minikube start`

Now deplyoment should have no errors.


Create the service:

`kubectl create -f ./kubernetes/postgres-service.yaml`

Create the database:

`kubectl get pods`

`kubectl exec <POD_NAME> --stdin --tty -- createdb -U sample todos`

## Node

Create deployment:

`kubectl create -f ./kubernetes/node-deployment-updated.yaml`


Create the service:

`kubectl create -f ./kubernetes/node-service.yaml`


Apply the migration and seed the database:

`kubectl get pods`
`kubectl exec <POD_NAME> knex migrate:latest`
`kubectl exec <POD_NAME> knex seed:run`

## Grab the external IP:

`minikube service node -n node`

It will give you addres like 127.0.0.1:$PORT

Test it out:

- [http://127.0.0.1:$PORT](http://127.0.0.1:$PORT)
- [http://127.0.0.1:$PORT/todos](http://127.0.0.1:$PORT/todos)


## Cleaning

`kubectl delete all --all -n node`
`kubectl delete pv NAME -n node`
`kubectl delete pvc NAME -n node`
