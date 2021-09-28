---
layout: post
title: Use Port Forwarding on K8S to Access Private Resources
categories: k8s
tag: k8s port-forward tunnel
author:
- Hai Nguyen
---
**The situation:** There is 1 MySQL Database instance, which only allow access from the K8S Cluster.

You need to connect to that database from your local machines. 

As you are working from home, set up a temporary rule on security group of the database is not a good idea.
## The solution

Create a tunnel on K8S Cluster.

This solution could be used for access to other private resources: PostgreSQL, Redis, MongoDB, Elasticsearch, etc. 
## How-to
### Prepare the environment variables

```
# I put specific values for my case
MYSQL_HOST=10.0.2.5
MYSQL_PORT=3306
TUNNEL_HOST=127.0.0.1
TUNNEL_PORT=33060
```
### Run tunnel pod

Run command:

```
kubectl run mysql-tunnel --image=alpine/socat -it --tty --rm --expose=true --port=${MYSQL_PORT} tcp-listen:${MYSQL_PORT},fork,reuseaddr tcp-connect:${MYSQL_HOST}:${MYSQL_PORT}
```

Open a new terminal to check the status of the tunnel pod & service

```
kubectl get pods
# stdout: 
# NAME           READY   STATUS    RESTARTS   AGE
# mysql-tunnel   1/1     Running   0          4m35s
kubectl get svc
# stdout:
# NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
# kubernetes     ClusterIP   10.254.0.1       <none>        443/TCP    4d16h
# mysql-tunnel   ClusterIP   10.254.107.165   <none>        3306/TCP   108m
```
### Run a port-forward to the service mysql-tunnel

Open a new terminal, run command

```
kubectl port-forward service/mysql-tunnel ${TUNNEL_PORT}:${MYSQL_PORT}
# stdout:
# Forwarding from 127.0.0.1:33060 -> 3306
# Forwarding from [::1]:33060 -> 3306
# Handling connection for 33060
```
### Test

with telnet

```
telnet ${TUNNEL_HOST} ${TUNNEL_PORT}
# stdout:
# Connected to localhost.
# Escape character is '^]'.
# _
# 5.7.31-0ubuntu0.18.04.1-log
# }jmoZAA(	[%ne_]mysql_native_passwor
```

or mysql client

```
mysql -h ${TUNNEL_HOST} -P ${TUNNEL_PORT} -u user_name -p database_name
```
