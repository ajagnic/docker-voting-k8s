# Docker-Voting in Kubernetes
A practice Kubernetes deployment that orchestrates the popular docker-compose example application.

Written with the intention that this would be an app used in production.

#### Author
Adrian Agnic [ [Github](https://github.com/ajagnic) ]

## Application Flow
![](voting-architecture.png)

#### Images and Resource Types
- [Traefik](https://hub.docker.com/_/traefik) - HTTP reverse-proxy (DaemonSet)
- [Postgres](https://hub.docker.com/_/postgres) - SQL database (StatefulSet)
- [Redis](https://hub.docker.com/_/redis) - Key-value data store (Deployment)
- [Vote](https://hub.docker.com/r/dockersamples/examplevotingapp_vote) - Python Flask web server (Deployment)
- [Worker](https://hub.docker.com/r/ajagnic/voting_fixed_worker) - Java process (modified version of the original [image](https://hub.docker.com/r/dockersamples/examplevotingapp_worker)) (Deployment)
- [Result](https://hub.docker.com/r/ajagnic/voting_fixed_result) - Node.js web server (modified version of the original [image](https://hub.docker.com/r/dockersamples/examplevotingapp_result)) (Deployment)

## Notes
- Ingress resource uses [nip.io](https://nip.io/) hostnames to simulate DNS resolution without modification of a _hosts_ file.

## Run
__Note__: This has only been tested as running on a local k8s cluster.

Create the IngressController before starting the application
```sh
kubectl apply -f ingress/traefik.yml
```

Create the application resources under the _voting_ namespace
```sh
kubectl apply -n voting -f .
```

Check that all resources were created successfully
```sh
kubectl get all -n ingress
```
```
NAME                READY   STATUS    RESTARTS   AGE
pod/traefik-x9f4g   1/1     Running   0          67s

NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/traefik         LoadBalancer   10.99.196.66     localhost     80:31794/TCP     85s
service/traefik-admin   NodePort       10.101.250.177   <none>        8080:32109/TCP   85s

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/traefik   1         1         1       1            1           <none>          85s
```

```sh
kubectl get all -n voting
```
```
NAME                          READY   STATUS    RESTARTS   AGE
pod/postgres-0                1/1     Running   0          18s
pod/redis-5c8c8c8dc5-rlj2l    1/1     Running   0          18s
pod/result-69d6c74d7c-74pch   1/1     Running   0          18s
pod/result-69d6c74d7c-qjlqw   1/1     Running   0          18s
pod/result-69d6c74d7c-rlfw9   1/1     Running   0          18s
pod/vote-74567fc67f-2cm9t     1/1     Running   0          17s
pod/vote-74567fc67f-dffh8     1/1     Running   0          18s
pod/vote-74567fc67f-qxqt9     1/1     Running   0          17s
pod/worker-76b678b674-988nd   1/1     Running   0          17s
pod/worker-76b678b674-drqbq   1/1     Running   0          17s

NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/db       ClusterIP   10.109.106.72    <none>        5432/TCP   18s
service/redis    ClusterIP   10.108.112.236   <none>        6379/TCP   18s
service/result   ClusterIP   10.107.191.14    <none>        80/TCP     18s
service/vote     ClusterIP   10.111.209.126   <none>        80/TCP     18s

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis    1/1     1            1           18s
deployment.apps/result   3/3     3            3           18s
deployment.apps/vote     3/3     3            3           18s
deployment.apps/worker   2/2     2            2           17s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/redis-5c8c8c8dc5    1         1         1       18s
replicaset.apps/result-69d6c74d7c   3         3         3       18s
replicaset.apps/vote-74567fc67f     3         3         3       18s
replicaset.apps/worker-76b678b674   2         2         2       17s

NAME                        READY   AGE
statefulset.apps/postgres   1/1     18s
```

## Use
__Local Deployment :__
- Vote - `http://vote.127.0.0.1.nip.io`
- Result - `http://result.127.0.0.1.nip.io`
- Traefik Admin - `http://{CLUSTER-IP}:{NODE-PORT}`

## Issues
- Redis Deployment cannot be replicated in current version
- Result Deployment is having issues with replication, possibly due to socket io implementation

## Incoming
- Adding LetsEncypt functionality to Traefik for HTTPS endpoints