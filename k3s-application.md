## Запускаем установку

```shell
curl -sfL https://get.k3s.io | sh -s - server --default-local-storage-path "/var/k3s/storage" --data-dir "/var/k3s/data" --disable=traefik
```

## Отключаем локальный storage

```shell
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```

```shell
curl -sfL https://get.k3s.io | sh -s - server --disable local-storage --disable=traefik
```
```shell
cat /var/lib/rancher/k3s/server/node-token
```
```shell
/etc/rancher/k3s/k3s.yaml
```

mkdir -p /etc/rancher/k3s/ && nvim /etc/rancher/k3s/config.yaml

write-kubeconfig-mode: "0644"
server: https://10.254.29.51:6443
token: K1066f79453898d0c93b0652c3edd86d647c5c535ee4fa8d29e865a77dc879fd41e::server:ee5836ac00cd0b9ee75715270a019951
tls-san:
  - k3s.radaev.su
cluster-init: true
disable:
- local-storage
- traefik

curl -sfL https://get.k3s.io | sh -

curl -sfL https://get.k3s.io | K3S_TOKEN=K107ca29fde1f39507e5079b53ae77a39862a5f4853f7f46d06e416fcf3e7129f98::server:abec0e56546e310a2dab269fba148279 sh -s - server --server https://10.254.29.51:6443 --disable local-storage --disable=traefik

curl -sfL https://get.k3s.io | K3S_URL=https://10.254.29.51:6443 K3S_TOKEN=K107ca29fde1f39507e5079b53ae77a39862a5f4853f7f46d06e416fcf3e7129f98::server:abec0e56546e310a2dab269fba148279 sh - 

curl -sfL https://get.k3s.io | K3S_TOKEN='l%TH]c4VvCT<Xj{' sh -s - server --cluster-init --disable local-storage --disable=traefik

curl -sfL https://get.k3s.io | K3S_TOKEN='l%TH]c4VvCT<Xj{' sh -s - server --server https://10.254.29.51:6443 --disable local-storage --disable=traefik


```shell
watch kubectl get pods -A
```

Не забудьте скопировать файл /etc/rancher/k3s/k3s.yaml на машины, где вы планируете обращаться к кластеру k3s

## Установка NFS
### From Lens

```shell
kubectl create ns nfs-provisioner
```

## Check VALUES BEFORE !!!

```shell
helm upgrade --install nfs-provisioner -n nfs-provisioner nfs-provisioner
```

## From ALL k3s-nodes
apt install -y nfs-common && systemctl enable rpcbind && systemctl start rpcbind

### Установка приложений

### Вспомогательные компоненты

## PriorityClass

```shell
kubectl apply -f 00-priorityclass.yaml
```

## Cert-manager

```shell
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.1/cert-manager.yaml
```

## CA и ClusterIssuer

Для работы нам потребуется свой CA и ClusterIssuer, который будет использован для подписи различных сертификатов.

```shell
kubectl apply -f CA/ca.yaml
```

## Импортируем сертификат CA в локальный файл.

kubectl -n cert-manager get secrets dev-ca -o jsonpath='{.data.ca\.crt}' | base64 -d > ca.crt

На машине, где запущен кластер k3s:
Добавление сертификата CA в Ubuntu

sudo mkdir /usr/local/share/ca-certificates/extra
sudo nvim /usr/local/share/ca-certificates/extra/dev-ca.crt
sudo update-ca-certificates


## Ingress controller

Устанавливаем чарт. Можно при помощи встроенного в k3s helm.

```shell
kubectl apply -f 01-ingress-controller.yaml
```

Проверяем:

```shell
kubectl -n ingress-nginx get all
```

```shell
curl http://10.254.29.50
```

## ArgoCD

```shell
kubectl create ns argo-cd
```

Создадим сикрет содержащий сертификат CA.

```shell
kubectl create secret generic -n argo-cd dev-ca-certs --from-file=dev-ca.pem=ca.crt
```

Запускаем ArgoCD

```shell
kubectl apply -f 02-argocd.yaml
```

## Reloader

https://github.com/stakater/Reloader

ArgoCD:

```shell
kubectl apply -f argocd-apps/reloader-app.yaml
```

или

```shell
kubectl apply -f manifests/reloader/
```

## PostgreSQL

Однонодовый PostgreSQL и pgadmin.

ArgoCD:

```shell
kubectl apply -f argocd-apps/postgre-app.yaml
```

или

```shell
kubectl create ns psql
kubectl -n psql apply -f manifests/psql/postgresql.yaml
```

## Redis

ArgoCD:

```shell
kubectl apply -f argocd-apps/redis-app.yaml
```

или

```shell
kubectl create ns redis
kubectl apply -f charts/redis.yaml
```

## Minio

ArgoCD:

```shell
kubectl apply -f argocd-apps/minio-app.yaml
```

или

```shell
kubectl apply -f charts/minio.yaml
```

## Minio console

ArgoCD:

```shell
kubectl apply -f argocd-apps/minio-console-app.yaml
```

или

```shell
kubectl -n minio apply -f manifests/minio-console/minio-console.yaml
```

## Mail relay

ArgoCD:

```shell
kubectl apply -f argocd-apps/mail-relay-app.yaml
```
или
```shell
kubectl create ns mail-relay
kubectl -n mail-relay apply -f manifests/mail-relay/
```

## Harbor

База данных harbor

ArgoCD:

```shell
kubectl apply -f argocd-apps/harbor-app.yaml
```

или:

```shell
kubectl create ns harbor
kubectl -n harbor apply -f charts/harbor.yaml
```

## Gitlab

Перед запуском GitLab требуется провести подготовительные действия.


```shell
kubectl create ns gitlab
```

Создаём сикреты, необходимые для работы gitlab:

```shell
kubectl -n gitlab apply -f gitlab-secrets
```

Создаём базу данных `gitlab` в PostgreSQL.

В minio создаём buckets:

- `gitlab-lfs-storage`
- `gitlab-artifacts-storage`
- `gitlab-uploads-storage`
- `gitlab-packages-storage`
- `gitlab-backup-storage`
- `gitlab-tmp-storage`

ArgoCD:

Почему то, в ArgoCD чарт не работает. Поэтому ставим через helm который встроен в k3s

```shell
kubectl apply -f charts/gitlab.yaml
```

## GitLab runner

В WEB интерфейсе создай runner. Получите токен и подставьте eго значение в Secret.

Tags
`global`

Description
Global runner for all projects

```shell
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: dev-gitlab-runner
  namespace: gitlab
  labels:
    manual: "yes"
type: Opaque
stringData:
  runner-registration-token: ""
  # тут подставляем полученный в WEB интерфейсе токен
  runner-token: "glrt-bX1SnA9b8Uh5r5KCSFsq"
  
  # S3 cache parameters
  accesskey: "admin"
  secretkey: "password"
EOF
```

В minio добавляем бакет `dev-runner-cache`

```shell
kubectl apply -f charts/gitlab-runner.yaml
```
test
