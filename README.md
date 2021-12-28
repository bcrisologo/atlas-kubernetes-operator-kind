# MongoDB Atlas Kubernetes Operator with Kind

## Summary

A tutorial that goes from deploying a local Kubernetes (K8) cluster that allows you to deploy a MongoDB Atlas cluster using the [Atlas Kubernetes Operator](https://www.mongodb.com/kubernetes/atlas-operator).  We will be using a kind cluster in this case for MacOS, but you can deploy a K8 cluster using other tools as well (i.e. OpenShift).

The tutorial below follows the deployment path from the [Introducing Atlas Kubernetes Operator](https://www.mongodb.com/blog/post/introducing-atlas-operator-kubernetes) blog and the [Quick Start official documentation](https://docs.atlas.mongodb.com/reference/atlas-operator/ak8so-quick-start/).

## Prerequisites
* A running Kubernetes cluster - in this case we'll be using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#configuring-your-kind-cluster).  *(Note: You would need also need [docker](https://docs.docker.com/engine/install/) installed and running.  Ensure that Kubernetes is enabled on it for `kind` to run)*.
* A [MongoDB Atlas Account](https://www.mongodb.com/atlas/database) and an [Atlas Project](https://docs.atlas.mongodb.com/tutorial/manage-projects/) created.
* [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Scope
This tutorial will go over deploying an M10 cluster to your Atlas project with a kubernetes cluster.  The additional options such as adding to the IP Access List and adding a Database User will not be included as the options can be added per the tutorials linked above.

## Process

### 1. Deploy the Kind Kubernetes cluster
First task is to deploy a `kind` cluster with the name `mongodb-atlas-system` first:

```
$ kind create cluster --name mongodb-atlas-system
```

### 2. Deploy the MongoDB Atlas Kubernetes Operator
Run the following command for the Atlas Kubernetes Operator to watch all namespaces in the cluster:
```
$ kubectl apply -f https://raw.githubusercontent.com/mongodb/mongodb-atlas-kubernetes/main/deploy/all-in-one.yaml
```

### 3. Create a secret with your API keys and organization ID

You would then need to create a secret for Kubernetes to call upon when connecting to your MongoDB Atlas Project.  In this case, we are putting it in the [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) `mongodb-atlas-system`:

```
$ kubectl create secret generic mongodb-atlas-operator-api-key \
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

### 4. Create the [AtlasProject](https://docs.atlas.mongodb.com/reference/atlas-operator/atlasproject-custom-resource/) custom resource

You will then need to create the custom resource for `atlascluster.atlas.mongodb.com` by running the command:
```
$ cat <<EOF | kubectl apply -f -
apiVersion: atlas.mongodb.com/v1
kind: AtlasProject
metadata:
  name: my-project
spec:
  name: <Reference Project>
EOF
```
where:
* `my-project` - designated reference name for the Kubernetes cluster to recognize (could be any but cannot use spaces in the name)
* `<Reference Project>` - project name found on MongoDB Atlas that you designate the cluster to be deployed

You can retrieve the project details by running the command:
```
kubectl get atlasprojects.atlas.mongodb.com -o yaml
```

### 5. Create the [AtlasCluster](https://docs.atlas.mongodb.com/reference/atlas-operator/atlascluster-custom-resource/#atlascluster-custom-resource) custom resource and deploy the Mongodb Atlas Cluster

In this section, you specify the cluster configuration
```
$ cat <<EOF | kubectl apply -f -
apiVersion: atlas.mongodb.com/v1
kind: AtlasCluster
metadata:
  name: my-atlas-cluster
spec:
  name: "Test-cluster"
  projectRef:
    name: my-project
  providerSettings:
    instanceSizeName: M10
    providerName: AWS
    regionName: US_EAST_1
EOF
```

Once run, you can retrieve details about the cluster by obtaining the custom resource details:
```
$ kubectl get atlasclusters.atlas.mongodb.com -o yaml
```

### (Optional) Delete Cluster

You can delete a cluster by indicating the resource `atlasclusters.atlas.mongodb.com` as per below:
```
$ kubectl delete atlasclusters.atlas.mongodb.com <spec-name-of-cluster>
```

## Footnote

You can visit the official [MongoDB Atlas Kubernetes Github Repo](https://github.com/mongodb/mongodb-atlas-kubernetes) as well.

For usage of the MongoDB Kubernetes Operator with MongoDB Ops Manager, you can visit [this documentation](https://docs.mongodb.com/kubernetes-operator/master/kind-quick-start/).
