# percona-server-mongodb-operator

[![Build Status](https://travis-ci.org/Percona-Lab/percona-server-mongodb-operator.svg?branch=master)](https://travis-ci.org/Percona-Lab/percona-server-mongodb-operator)
[![Go Report Card](https://goreportcard.com/badge/github.com/Percona-Lab/percona-server-mongodb-operator)](https://goreportcard.com/report/github.com/Percona-Lab/percona-server-mongodb-operator)
[![codecov](https://codecov.io/gh/Percona-Lab/percona-server-mongodb-operator/branch/master/graph/badge.svg)](https://codecov.io/gh/Percona-Lab/percona-server-mongodb-operator)

A Kubernetes operator for [Percona Server for MongoDB](https://www.percona.com/software/mongo-database/percona-server-for-mongodb) based on the [Operator SDK](https://github.com/operator-framework/operator-sdk).

# DISCLAIMER

**This code is incomplete, expect major issues and changes until this repo has stabilised!**

# Run

## Requirements

This code was developed/tested for Kubernetes version 1.10 to 1.11 only!

## Run the Operator
1. Add the 'psmdb' Namespace to Kubernetes:
    ```
    kubectl create namespace psmdb
    kubectl config set-context $(kubectl config current-context) --namespace=psmdb
    ```
1. Add the MongoDB Users secrets to Kubernetes. **Update mongodb-users.yaml with new passwords!!!**
    ```
    kubectl create -f deploy/mongodb-users.yaml
    ```
 
1. Extra step (for Google Kubernetes Engine ONLY!!!)
    ```
    kubectl create clusterrolebinding cluster-admin-binding1 --clusterrole=cluster-admin --user=<myname@example.org>
    ```
1. Start the percona-server-mongodb-operator within Kubernetes:
    ```
    kubectl create -f deploy/rbac.yaml
    kubectl create -f deploy/crd.yaml
    kubectl create -f deploy/operator.yaml
    ```
1. Create the Percona Server for MongoDB cluster:
    ```
    kubectl apply -f deploy/cr.yaml
    ```
1. Wait for the operator and replica set pod reach Running state:
    ```
    $ kubectl get pods
    NAME                                               READY   STATUS    RESTARTS   AGE
    my-cluster-name-rs0-0                              1/1     Running   0          8m
    my-cluster-name-rs0-1                              1/1     Running   0          8m
    my-cluster-name-rs0-2                              1/1     Running   0          7m
    percona-server-mongodb-operator-754846f95d-sf6h6   1/1     Running   0          9m
    ``` 
1. From a *'mongo'* shell add a [readWrite](https://docs.mongodb.com/manual/reference/built-in-roles/#readWrite) user for use with an application *(hostname/replicaSet in mongo uri may vary for your situation)*:
    ```
    $ kubectl run -i --rm --tty percona-client --image=percona/percona-server-mongodb:3.6 --restart=Never -- bash -il
    mongodb@percona-client:/$ mongo mongodb+srv://userAdmin:admin123456@my-cluster-name.psmdb.svc.cluster.local/admin?replicaSet=rs0
    rs0:PRIMARY> db.createUser({
        user: "app",
        pwd: "myAppPassword",
        roles: [
          { db: "myApp", role: "readWrite" }
        ]
    })
    Successfully added user: {
    	"user" : "app",
    	"roles" : [
    		{
    			"db" : "myApp",
    			"role" : "readWrite"
    		}
    	]
    }
    ```
1. Again from a *'mongo'* shell, insert and retrieve a test document in the *'myApp'* database as the new application user:
    ```
    $ kubectl run -i --rm --tty percona-client --image=percona/percona-server-mongodb:3.6 --restart=Never -- bash -il
    mongodb@percona-client:/$ mongo mongodb+srv://myApp:myAppPassword@my-cluster-name.psmdb.svc.cluster.local/admin?replicaSet=rs0
    rs0:PRIMARY> use myApp
    switched to db myApp
    rs0:PRIMARY> db.test.insert({ x: 1 })
    WriteResult({ "nInserted" : 1 })
    rs0:PRIMARY> db.test.findOne()
    { "_id" : ObjectId("5bc74ef05c0ec73be760fcf9"), "x" : 1 }
    ```
