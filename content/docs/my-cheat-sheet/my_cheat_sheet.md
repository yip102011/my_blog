---
title: Other Cheat Sheet
date: "2022-10-10"
tags:
  - cheat-sheet
aliases:
  - my-other-cheat-sheet
---

My other cheat Sheet

<!--more-->

## Kubernetes

- [official cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [kubectl](https://jamesdefabia.github.io/docs/user-guide/kubectl/kubectl/)
- [go template url](https://pkg.go.dev/text/template#hdr-Text_and_spaces)

```bash
# kubectl exec one of deployment pod
kubectl exec -it deploy/my-app -n bes -- bash

# kubectl get all resource includeing custom resource
kubectl get $(kubectl api-resources --namespaced=true --no-headers -o name | grep -v -E 'events|bindings$|localsubjectaccessreviews' | paste -s -d, - )

# kubectl show pod cpu and memory config of first container
kubectl get pod -o custom-columns=POD_NAME:.metadata.name,CONTAINER:.spec.containers[0].name,CPU_MIN:.spec.containers[0].resources.limits.cpu,CPU_MAX:.spec.containers[0].resources.requests.cpu,MEM_MAX:.spec.containers[0].resources.limits.memory,MEM_MIN:.spec.containers[0].resources.requests.memory,STATUS:.status.phase

# kubectl list all nodeport
kubectl get svc --all-namespaces -o go-template='{{range .items}}{{ $svc := . }}{{range.spec.ports}}{{if .nodePort}}{{.nodePort}}{{","}}{{if .name}}{{printf "%-10s" .name}}{{else}}{{printf "%-10s" ""}}{{end}}{{","}}{{$svc.metadata.namespace}}{{","}}{{$svc.metadata.name}}{{"\n"}}{{end}}{{end}}{{end}}'

# create nodeport svc
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: my-svc
  namespace: default
spec:
  ports:
    - port: 80
      nodePort: 30002
  selector:
    app: my-app
  type: NodePort
EOF

# backup etcd data
sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt \
  --key=/etc/kubernetes/pki/apiserver-etcd-client.key \
  snapshot save ~/etcd_backup
  
# view container log
sudo docker logs -n 100 -f $(sudo docker ps -f name=k8s_kube-vip -q)
sudo docker logs -n 100 -f $(sudo docker ps -f name=k8s_etcd -q)
sudo docker logs -n 100 -f $(sudo docker ps -f name=k8s_kube-apiserver -q)
sudo docker logs -n 100 -f $(sudo docker ps -f name=k8s_kube-scheduler -q)
sudo docker logs -n 100 -f $(sudo docker ps -f name=k8s_kube-controller-manager -q)
```

## Helm

```bash
# download helm chart
helm pull bitnami/redis --untar --untardir ./redis-helm-charts

# render helm template
helm template my-redis bitnami/redis --output-dir=./otuput-dir --dry-run -f="my-values.yaml"

# ls installed app
helm ls -A

# get installed values
helm get values my-redis -n redis
```

## ArgoCD

Ref: https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd/

```bash
# get init admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# create container
docker run --rm -it argoproj/argocd

# login
argocd login "192.168.0.1" --insecure --username "isaac" --password "xxxxxx"

# restart deployment
argocd app actions run "my-app" restart --kind Deployment

# update password
argocd account update-password --account admin

# set app param
kubectl patch Application --type=merge -n=argocd -p '{"spec":{"source":{"helm":{"parameters":[{"name":"replicaCount","value":"0"}]}}}}' my-app

# set app param2
kubectl exec deploy/argocd-server -n argocd -- bash -c "argocd login 127.0.0.1:8080 --insecure --username admin --password 'XXXXXX'"
kubectl exec deploy/argocd-server -n argocd -- bash -c "argocd app set my-app -p replicaCount=0"
```

## Docker

```bash
# remove image
docker image rm my-reg/my-image:latest

# remove image with filter
docker image rm $(docker images --filter=reference='my-reg/*' --format "{{.Repository}}:{{.Tag}}")

# add new tag
docker tag "my-reg/my-image:latest" "my-reg/my-image:new-tag"

# docker save image as file
docker save -o "my-image.tar" "my-reg/my-image:latest"

# docker load image file
docker load --input "my-image.tar"
```

## mongodb

```bash
# backup database to file
mongodump --uri="mongodb://admin:XXXXXW@192.168.0.101:10001,192.168.0.102:10002,192.168.0.103:10003/?authSource=admin&replicaSet=my_replica_set&readPreference=primary" --out=mongodump/ --db=my_db

# restore database from file
mongorestore --uri="mongodb://admin:XXXXXW@192.168.0.101:10001,192.168.0.102:10002,192.168.0.103:10003/?authSource=admin&replicaSet=my_replica_set&readPreference=primary" --db=my_db mongodump/my_db
```
