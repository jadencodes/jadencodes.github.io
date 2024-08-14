---
layout: post
title:  "CV CTF - Siege of the Fortress Keep"
date:   2024-08-14 00:10:20 -0600
categories: ctf cloudvillage
tags: defcon ctf cloudvillage
description: "Writeup/Solution to Defcon 2024 Cloud Village CTF: Siege of the Fortress Keep"
---

This was probably my personal favorite. It was extremely unique and challenging.

## Prompt

> In a labyrinthine fortress besieged by an army of sorcerers. Amid the chaos, countless doors appear, each guarded by a fake account or user posing as an ally or foe. Your task is to decipher the true path through this container setup, identifying genuine setups to unlock the fortress keep and claim victory. You will find dungeon at - CONJURE ME

`siege-the-fortress.tar`

## Solution

Untarring the file gave us `docker-compose.yaml` and `siege-image.tar`.

The docker-compose file was very basic:

```yaml
version: '3.8'

services:
  python_app:
    image: siege-image:latest
    container_name: lilian
    command: tail -f /dev/null
```

So next step was to load the `siege-image.tar` and run the docker container:

```console
jaden@monstera:~/defcon/dc32/cv/seige-the-fortress  $ docker load < siege-image.tar 
Loaded image: siege-image:latest

 $ docker run --rm -it siege-image:latest bash
root@47c4a0add966:/app# ls
siege_the_fortress
root@47c4a0add966:/app# ./siege_the_fortress 
I miss minikube. Do you?
root@47c4a0add966:/app# 
```

Interesting... [minikube](https://minikube.sigs.k8s.io/docs/start/) is a tool for running kubernetes locally. It handles all the controller pods setup and networking. We have used it in the past for local development. I spun it up with extra verbose (`-v=8`).

Turns out, you can load images into minikube and then run them as pods!

```console
$ kubectl run siege --image=siege-image:latest --image-pull-policy=Never
pod/siege created

$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
siege   1/1     Running   0          7s

```

Excecing in we can try and run the binary again:
```console
$ kubectl exec -it siege -- bash
root@siege:/app# ls
siege_the_fortress
root@siege:/app# ./siege_the_fortress 
You got the idea - but may you need to be more than just a player - maybe master role?
```

So now it is clearly interacting with the kubernetes api. We can create a role named master with access to everything on the cluster in a yaml file. We also define the pod here so we can easily assign it the role we make:

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: master
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: master
  namespace: default
subjects:
- kind: ServiceAccount
  name: master
  namespace: default
roleRef:
  kind: Role
  name: master
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: master
  namespace: default
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-08-10T00:13:25Z"
  labels:
    run: siege
  name: lilian
  namespace: default
spec:
  containers:
  - image: siege-image:latest
    imagePullPolicy: Never
    name: lilian
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccountName: master
  serviceAccount: master
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-j88c5
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
```

**NOTE:** I also renamed the pod and container to lilian since that was in the docker-compose.yaml file, unsure if it was needed though.

Execing in and trying again:

```console
$ kubectl exec -it lilian -- bash
root@lilian:/app# ls
siege_the_fortress
root@lilian:/app# ./siege_the_fortress 
.....................
A spell has been casted
Look around
.....................
name?
```

We got further! It is now requesting a `name` input. It also tells us to look around.

Looking around we found a deployment and single pod created:

```console
$ kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
dungeon-room-69bbb98b9c-g5l8z   1/1     Running   0          2m14s
lilian                          1/1     Running   0          2m50s

$ kubectl get deployments
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
dungeon-room   1/1     1            1           2m19s
```

After a lot of trial and error, I gave the original pod the _name_ of our uniquely created `dungeon-room` pod. In this case `dungeon-room-69bbb98b9c-g5l8z`.

We get a different error message:

```console
root@lilian:/app# ./siege_the_fortress 
.....................
A spell has been casted
Look around
.....................
name? dungeon-room-69bbb98b9c-g5l8z
Key not in the right door in the right room


Try again


name?
```

Looking at the kube-system api server logs (because we started them in verbose mode) we can see what the siege/lilian pod were doing:

```console
"HTTP" verb="CONNECT" URI="/api/v1/namespaces/default/pods/dungeon-room-69bbb98b9c-g5l8z/exec?command=cat&command=%2Froot%2Fflag.txt&container=drogon&stderr=true&stdout=true" latency="48.175224ms" userAgent="kubectl/v1.30.3 (linux/amd64) kubernetes/6fc0a69" audit-ID="14b72fcc-2ab5-4206-b937-3ed2d3fa235c" srcIP="10.244.0.6:56670" hijacked=true
```

We can see here that our pod with the binary in it `siege_the_fortress` is making a call to the kubernetes pod it created (the dungeon-room pod) and running `cat /root/flag.txt`.

From inside the dungeon pod,  we can touch that file:

```console
root@dungeon-room-69bbb98b9c-g5l8z:/# touch /root/flag.txt
```

Then back in the original pod, again give the dungeon-room name and it _looks_ like we have the flag!

```console
root@lilian:/app# ./siege_the_fortress 
name? dungeon-room-69bbb98b9c-g5l8z


You have conquered the dungeon!!


FLAG-{d3ng3ond41d8cd98f00b204e9800998e}
```

HOWEVER, after trying to submit this flag it said it was incorrect... Taking a look in the dungeon room pod for more clues gives us an interesting file in one of the home directories:

```console
root@dungeon-room-69bbb98b9c-g5l8z:/home# tree
.
|-- !
|-- cloud
|   `-- flag.txt
|-- fun
|-- is
|-- ubuntu
|-- user1
|-- user10
|-- user11
|-- user12
|-- user13
|-- user14
|-- user15
```

Inside `/home/cloud/flag.txt` we see:

```plaintext
This maybe useful somewhere
figure out the right door
in the dungeons!
```

Now we copy that flag from `/home/cloud/flag.txt` to `/root/flag.txt` and run the dungeon again in the original pod:

```console
root@lilian:/app# ./siege_the_fortress 
Hmmm.. So you have been here already.
name? dungeon-room-69bbb98b9c-g5l8z


You have conquered the dungeon!!


FLAG-{d3ng3on38f249e1cabd66027841fb1ee}
root@lilian:/app#
```

This flag worked!! That was a very unique challenge and lots of fun.
