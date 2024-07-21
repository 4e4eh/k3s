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
sudo vim /usr/local/share/ca-certificates/extra/dev-ca.crt


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