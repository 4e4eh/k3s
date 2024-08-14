curl -sfL https://get.k3s.io | sh -s - server --default-local-storage-path "/var/k3s/storage" --data-dir "/var/k3s/data" --disable=traefik

Отключаем локальный storage
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'


curl -sfL https://get.k3s.io | sh -s - server --disable local-storage --disable=traefik
cat /var/lib/rancher/k3s/server/node-token
/etc/rancher/k3s/k3s.yaml

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

curl -sfL https://get.k3s.io | K3S_TOKEN='l%TH]c4VvCT<Xj{' sh -s - server --cluster-init --disable local-storage --disable=traefik --tls-san=10.254.29.51

curl -sfL https://get.k3s.io | K3S_TOKEN='l%TH]c4VvCT<Xj{' sh -s - server --server https://10.254.29.51:6443 --disable local-storage --disable=traefik --tls-san=10.254.29.51


token: K10717821a205f68fed3c77b42190502b387f67806bc776e5ea3f1d7cae772773b7::server:l%TH]c4VvCT<Xj{

watch kubectl get pods -A
Не забудьте скопировать файл /etc/rancher/k3s/k3s.yaml на машины, где вы планируете обращаться к кластеру k3s



mkdir -p /etc/rancher/k3s

echo "K1066f79453898d0c93b0652c3edd86d647c5c535ee4fa8d29e865a77dc879fd41e::server:ee5836ac00cd0b9ee75715270a019951" > /etc/rancher/k3s/cluster-token

nvim /etc/rancher/k3s/kubelet.config

apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
shutdownGracePeriod: 30s
shutdownGracePeriodCriticalPods: 10s

nvim /etc/rancher/k3s/config.yaml

token-file: /etc/rancher/k3s/cluster-token
disable:
  - local-storage
  - servicelb
  - traefik
tls-san:
  - 10.254.29.51
  - k3s.radaev.su
kubelet-arg:
  - config=/etc/rancher/k3s/kubelet.config
write-kubeconfig-mode: 644

curl -sfL https://get.k3s.io | sh -s - server --cluster-init

curl -sfL https://get.k3s.io | sh -s - server --cluster-init --tls-san=10.254.29.51 --disable local-storage --disable traefik

curl -sfL https://get.k3s.io | sh -s - server --server https://10.254.29.51:6443 --tls-san=10.254.29.51 --disable local-storage --disable traefik

watch kubectl get pods -A


curl -sfL https://get.k3s.io | sh -s - server --server https://10.254.29.51:6443





mkdir -p /etc/rancher/k3s

echo "K10717821a205f68fed3c77b42190502b387f67806bc776e5ea3f1d7cae772773b7::server:l%TH]c4VvCT<Xj{" > /etc/rancher/k3s/cluster-token

write-kubeconfig-mode: 644
token-file: /etc/rancher/k3s/cluster-token
disable:
  - local-storage
  - traefik
tls-san:
  - 10.254.29.51
  - k3s.radaev.su
cluster-init: true

curl -sfL https://get.k3s.io | sh -s - server




nvim /etc/rancher/k3s/config.yaml

write-kubeconfig-mode: 644    
server: https://10.254.29.51:6443
token: K1097d26870fce960e66c96ea45f1b317e831e03a4f2e38e2d0b900d5b556fd6ced::server:36f92398855e7e27e756a4088ade1bb4    
disable:    
  - local-storage    
  - traefik    
tls-san:    
  - 10.254.29.51    
  - k3s.radaev.su

  curl -sfL https://get.k3s.io | sh -