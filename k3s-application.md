curl -sfL https://get.k3s.io | sh -s - server --default-local-storage-path "/var/k3s/storage" --data-dir "/var/k3s/data" --disable=traefik

Отключаем локальный storage
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'


curl -sfL https://get.k3s.io | sh -s - server --disable local-storage --disable=traefik

Установка приложений

Вспомогательные компоненты

PriorityClass

kubectl apply -f 00-priorityclass.yaml

Cert-manager

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.1/cert-manager.yaml

CA и ClusterIssuer

Для работы нам потребуется свой CA и ClusterIssuer, который будет использован для подписи различных сертификатов.

kubectl apply -f CA/ca.yaml

Импортируем сертификат CA в локальный файл.

kubectl -n cert-manager get secrets dev-ca -o jsonpath='{.data.ca\.crt}' | base64 -d > ca.crt

На машине, где запущен кластер k3s:
Добавление сертификата CA в Ubuntu

sudo mkdir /usr/local/share/ca-certificates/extra
sudo nvim /usr/local/share/ca-certificates/extra/dev-ca.crt
sudo update-ca-certificates


Ingress controller

Устанавливаем чарт. Можно при помощи встроенного в k3s helm.

kubectl apply -f 01-ingress-controller.yaml
Проверяем:

kubectl -n ingress-nginx get all
curl http://10.254.29.50

ArgoCD

kubectl create ns argo-cd
Создадим сикрет содержащий сертификат CA.

kubectl create secret generic -n argo-cd dev-ca-certs --from-file=dev-ca.pem=ca.crt
Запускаем ArgoCD

kubectl apply -f 02-argocd.yaml

Reloader

https://github.com/stakater/Reloader

ArgoCD:

kubectl apply -f argocd-apps/reloader-app.yaml
или

kubectl apply -f manifests/reloader/

PostgreSQL

Однонодовый PostgreSQL и pgadmin.

ArgoCD:

kubectl apply -f argocd-apps/postgre-app.yaml
или

kubectl create ns psql
kubectl -n psql apply -f manifests/psql/postgresql.yaml

Redis

ArgoCD:

kubectl apply -f argocd-apps/redis-app.yaml
или

kubectl create ns redis
kubectl apply -f charts/redis.yaml

Minio

ArgoCD:

kubectl apply -f argocd-apps/minio-app.yaml
или

kubectl apply -f charts/minio.yaml

Minio console

ArgoCD:

kubectl apply -f argocd-apps/minio-console-app.yaml
или

kubectl -n minio apply -f manifests/minio-console/minio-console.yaml

Mail relay

ArgoCD:

kubectl apply -f argocd-apps/mail-relay-app.yaml
или

kubectl create ns mail-relay
kubectl -n mail-relay apply -f manifests/mail-relay/

Harbor

База данных harbor

ArgoCD:

kubectl apply -f argocd-apps/harbor-app.yaml
или:

kubectl create ns harbor
kubectl -n harbor apply -f charts/harbor.yaml

Gitlab

Перед запуском GitLab требуется провести подготовительные действия.

kubectl create ns gitlab
Создаём сикреты, необходимые для работы gitlab:

kubectl -n gitlab apply -f gitlab-secrets
Создаём базу данных gitlab в PostgreSQL.

В minio создаём buckets:

gitlab-lfs-storage
gitlab-artifacts-storage
gitlab-uploads-storage
gitlab-packages-storage
gitlab-backup-storage
gitlab-tmp-storage

ArgoCD:

Почему то, в ArgoCD чарт не работает. Поэтому ставим через helm который встроен в k3s

kubectl apply -f charts/gitlab.yaml

GitLab runner

В WEB интерфейсе создай runner. Получите токен и подставьте eго значение в Secret.

Tags
global

Description
Global runner for all projects

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
В minio добавляем бакет dev-runner-cache.

kubectl apply -f charts/gitlab-runner.yaml

