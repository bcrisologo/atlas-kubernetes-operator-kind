# MongoDB Atlas Kubernetes Operator with Kind

## Summary

A tutorial that goes from deploying a local Kubernetes (K8) cluster that allows you to deploy a MongoDB Atlas cluster using the [Atlas Kubernetes Operator](https://www.mongodb.com/kubernetes/atlas-operator).  We will be using a kind cluster in this case for MacOS, but you can deploy a K8 cluster using other tools as well (i.e. OpenShift).

The tutorial below follows the deployment path from the [Introducing Atlas Kubernetes Operator](https://www.mongodb.com/blog/post/introducing-atlas-operator-kubernetes) blog and the [Quick Start official documentation](https://docs.atlas.mongodb.com/reference/atlas-operator/ak8so-quick-start/).

## Prerequisites
* A running Kubernetes cluster - in this case we'll be using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#configuring-your-kind-cluster).  You would need also need [docker](https://docs.docker.com/engine/install/) installed and running.  Ensure that Kubernetes is enabled on it for `kind` to run.
* A [MongoDB Atlas Account](https://www.mongodb.com/atlas/database) and an [Atlas Project](https://docs.atlas.mongodb.com/tutorial/manage-projects/) created.
* [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Scope
This tutorial will go over deploying an M10 cluster to your Atlas project with a kubernetes cluster.  The additional options such as adding to the IP Access List and adding a Database User will not be included as the options can be added per the tutorials linked above.

## Process

### 1. Deploy the Kind Kubernetes cluster
First task is to deploy a `kind` cluster with the name `mongodb-atlas-system` first:

```bash
kind create cluster --name mongodb-atlas-system
```

### 2. Deploy the MongoDB Atlas Kubernetes Operator
Run the following command for the Atlas Kubernetes Operator to watch all namespaces in the cluster:
```
kubectl apply -f https://raw.githubusercontent.com/mongodb/mongodb-atlas-kubernetes/main/deploy/all-in-one.yaml
```

### 3. Create a secret with your API keys and organization ID

You would then need to create a secret for Kubernetes to call upon when connecting to your MongoDB Atlas Project.

```
kubectl create secret generic mongodb-atlas-operator-api-key \
    --from-literal="orgId=<atlas_organization_id>" \
    --from-literal="publicApiKey=<atlas_api_public_key>" \
    --from-literal="privateApiKey=<atlas_api_private_key>" \
    -n mongodb-atlas-system
```

You will need to obtain the Organization ID found on the URL of the MongoDB Atlas Organization page similar to below:
```
https://cloud.mongodb.com/v2#/org/<Org ID>/projects
```
The API keys should be created via the MongoDB Atlas Project API level.  You can read more on the [Atlas API Administration documentation](https://docs.atlas.mongodb.com/api/atlas-admin-api/).

## Footnote

You can visit the official [MongoDB Atlas Kubernetes Github Repo](https://github.com/mongodb/mongodb-atlas-kubernetes) as well.

For usage of the MongoDB Kubernetes Operator with MongoDB Ops Manager, you can visit [this documentation](https://docs.mongodb.com/kubernetes-operator/master/kind-quick-start/).
