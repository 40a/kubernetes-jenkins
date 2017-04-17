Running Jenkins master and slaves in a Kubernetes cluster
=========================================================

Kubernetes examples running Jenkins master and slaves

Creating a cluster
==================

Local with Docker Compose
-------------------------

A local testing cluster with one node can be created with Docker Compose

```
docker-compose up
```

When using boot2docker or Docker Engine with a remote host, the remote Kubernetes API can be exposed
with `docker-machine ssh MACHINE_NAME -L 0.0.0.0:8080:localhost:8080` or `boot2docker ssh -L 0.0.0.0:8080:localhost:8080`

More info

* [Docker CookBook examples](https://github.com/how2dock/docbook/tree/master/ch05/docker)
* [Kubernetes Getting started with Docker](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/docker.md)

Google Compute Engine
---------------------

```
export KUBERNETES_HOME=~/kubernetes
export KUBERNETES_PROVIDER=gce
export KUBERNETES_NUM_MINIONS=2
export KUBE_GCE_ZONE=us-central1-a
$KUBERNETES_HOME/cluster/kube-up.sh
```

Creating the pods and services
==============================

GKE
-------

```
gcloud compute disks create --size 20GB jenkins-data-disk
kubectl get nodes
kubectl create -f jenkins-master-gke.yml
kubectl get rc
kubectl get pods
kubectl create -f service-gke.yml
kubectl get services
kubectl create -f jenkins-slaves.yml
kubectl get rc
kubectl get pods
kubectl scale replicationcontrollers --replicas=2 jenkins-slave
kubectl describe services/jenkins
gcloud compute forwarding-rules list
```

AWS
-------
This assumes a working kubernetes installation. I generate mine with [kops](https://github.com/kubernetes/kops).
If `kubectl cluster-info` gives you output about the location of the API server, you are likely in pretty good shape.
Next, create the working volume for jenkins:
`aws ec2 create-volume --availability-zone us-east-1a --size 20 --volume-type gp2

You'll get a response back that looks something like this:
```
{
    "AvailabilityZone": "us-east-1a",
    "Encrypted": false,
    "VolumeType": "gp2",
    "VolumeId": "vol-002d2b99000000000", # Write this value down
    "State": "creating",
    "Iops": 100,
    "SnapshotId": "",
    "CreateTime": "2016-12-24T17:39:34.725Z",
    "Size": 20
}
```
Edit `jenkins-master-aws.yml` and put the VolumeID in the volumeID field`.

```
kubectl create -f jenkins-master-aws.yml
kubectl get rc
kubectl get pods
kubectl create -f service-aws.yml
kubectl get services
kubectl describe service jenkins
kubectl create -f jenkins-slaves.yml
kubectl get rc
kubectl get pods
kubectl scale replicationcontrollers --replicas=2 jenkins-slave
kubectl describe services/jenkins
```
These instructions get you a publically accessible Jenkins dashboard at the load balancer specified in `kubectl describe service jenkins`. This is likely not ideal for a production environment for a number of reasons to be explored at some future date.

Vagrant
-------

```
kubectl get nodes
kubectl create -f jenkins-master-vagrant.yml
kubectl get rc
kubectl get pods
kubectl create -f service-vagrant.yml
kubectl get services
kubectl describe services/jenkins
kubectl create -f jenkins-slaves.yml
kubectl get rc
kubectl get pods
kubectl scale replicationcontrollers --replicas=2 jenkins-slave
```


Rolling update
==============

```
kubectl rolling-update jenkins-slave --update-period=10s -f replication-v2.yml
```

Tearing down
============

```
kubectl stop replicationcontrollers jenkins-slave
kubectl stop replicationcontrollers jenkins
kubectl delete services jenkins
$KUBERNETES_HOME/cluster/kube-down.sh
```

Demo
====

Kubernetes cluster up
[![asciicast](https://asciinema.org/a/18161.png)](https://asciinema.org/a/18161)

Jenkins master and slaves provisioning
[![asciicast](https://asciinema.org/a/18162.png)](https://asciinema.org/a/18162)

Kubernetes cluster teardown
[![asciicast](https://asciinema.org/a/18163.png)](https://asciinema.org/a/18163)
