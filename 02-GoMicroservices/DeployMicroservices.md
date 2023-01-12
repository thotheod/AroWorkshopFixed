# LAB 1: Go Microservices Walkthrough

## 2.1 Application Overview
check [changelog](changelog.md) for error in table

## 2.2 Create Cluster
Done here [DeployAro.azcli](../01-DeployAro/deployAro.azcli)

## 2.3 Create Project

### Login to web console

1. run `az aro list -o table` to get the URL of the web console
2. then from the web console you can select "copy login command"and take the token to login with OC CLI as below (alternatively you can login with username/password as described in [deployAro](../01-DeployAro/deployAro.azcli) last lines )
    - oc login --token=sha256_XYZXYZXYZXYZ_ --server=https://api.dn3nxtdh.northeurope.aroapp.io:6443

### create a project

```bash
# Create a new project
oc new-project workshop

# delete project (if needed to cleanup)
# oc delete project workshop
```

## 2.4 Deploy MongoDB

``` bash
oc new-app bitnami/mongodb \
  -e MONGODB_USERNAME=ratingsuser \
  -e MONGODB_PASSWORD=XYZ \
  -e MONGODB_DATABASE=ratingsdb \
  -e MONGODB_ROOT_USER=root \
  -e MONGODB_ROOT_PASSWORD=XYZ

# Verify if the mongoDB pod was created successfully
oc get all

# Retrieve mongoDB service hostname
oc get svc mongodb
# internal DNS: mongodb.workshop.svc.cluster.local
# is formed of [service name].[project name].svc.cluster.local. This resolves only within the cluster.

```

## 2.5 Deploy Ratings API
fork / copy the application
my URL is https://github.com/thotheod/mslearn-aks-workshop-ratings-api
and https://github.com/thotheod/mslearn-aks-workshop-ratings-web

``` bash
# to create the app
oc new-app https://github.com/thotheod/mslearn-aks-workshop-ratings-api --strategy=source --name=rating-api 

# to dry run the new app:
# oc new-app https://github.com/thotheod/mslearn-aks-workshop-ratings-api --strategy=source --name=rating-api -o yaml > ratings-api-DryRun.yaml

# to delete the app
# oc delete all -l app=rating-api

# configure ENV variable for the app to connect to MongoDB: MONGODB_URI environment variable. This URI should look like mongodb://ratingsuser:XYZ@mongodb.workshop.svc.cluster.local:27017/ratingsdb
oc set env deploy/rating-api MONGODB_URI=mongodb://ratingsuser:XYZ@mongodb.workshop.svc.cluster.local:27017/ratingsdb

```

### Setup GitHub webhook

``` bash
# get secret
oc get bc/rating-api -o=jsonpath='{.spec.triggers..github.secret}'
oc describe bc/rating-api
# https://api.dn3nxtdh.northeurope.aroapp.io:6443/apis/build.openshift.io/v1/namespaces/workshop/buildconfigs/rating-api/webhooks/<secret>/github
``` 


## 2.6 Deploy Ratings frontend

``` bash
# Build and deploy rating-web
oc new-app https://github.com/thotheod/mslearn-aks-workshop-ratings-web --strategy=docker --name=rating-web

# configure ENV CLI
oc set env deploy rating-web API=http://rating-api:8080
#  oc set env deploy rating-web API=http://rating-api.workshop.svc.cluster.local:8080

# Expose the rating-web service using a Route
oc expose svc/rating-web
oc get route rating-web 

```

## Fix Rating API Svc
we need to change the service. The rating-API SVC by default exposes rating-API pod's port 8080 to 8080, which is wrong. The pod is exposing port 3000.
so we need to change the service yaml, change the targetport from 8080 to 3000

### Setup GitHub webhook for ratingWeb
``` bash
# Setup GitHub webhook
oc get bc/rating-web -o=jsonpath='{.spec.triggers..github.secret}'

oc describe bc/rating-web
#  https://api.dn3nxtdh.northeurope.aroapp.io:6443/apis/build.openshift.io/v1/namespaces/workshop/buildconfigs/rating-web/webhooks/secret/github
```