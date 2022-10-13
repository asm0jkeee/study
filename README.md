# Astra Linux Special Edition 1.7 (Smolensk) и кластер Kubernetes 

## Установка Astra Linux SE 1.7 

Установка выполняется стандартно, как любой дистрибутив Linux.

## Установка кластера с MSA Platform

Не хватает chrony
Добавить репозиторий
```
deb http://ftp.de.debian.org/debian buster main
```
И добавить PUB_KEY
```
apt-key adv --recv-key --keyserver pgpkeys.mit.edu {PUB_KEY}
```

Установить chrony
```
sudo apt  install -y chrony
```

## Установка кластера вручную

1. containerd установлен, работает
2. установка kubeadm, kubectl и kubelet 
При попытке установки выдает ошибку:
```
sudo apt-get install -y kubelet kubeadm kubectl
Чтение списков пакетов… Готово
Построение дерева зависимостей
Чтение информации о состоянии… Готово
Некоторые пакеты не могут быть установлены. Возможно, то, что вы просите,
неосуществимо, или же вы используете нестабильную версию дистрибутива, где
запрошенные вами пакеты ещё не созданы или были удалены из Incoming.
Следующая информация, возможно, вам поможет:

Следующие пакеты имеют неудовлетворённые зависимости:
 kubelet : Зависит: ebtables но он не может быть установлен
           Зависит: conntrack но он не может быть установлен
E: Невозможно исправить ошибки: у вас зафиксированы сломанные пакеты.
```

Решение:
1. добавить репозиторий 
```
deb http://cz.archive.ubuntu.com/ubuntu bionic main 
```

2. попробовать сделать apt-get update и в выводе получить ключ {PUB_KEY}, которого не хватает для подписи репозитория
3. выполнить команду

```
apt-key adv --recv-key --keyserver pgpkeys.mit.edu {PUB_KEY}
```

### Запуск kubeadm init

Выдает ошибки:
```
kubeadm init --pod-network-cidr=10.19.0.0/16
[init] Using Kubernetes version: v1.25.2
[preflight] Running pre-flight checks
        [WARNING Swap]: swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
        [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

Решение:
[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
1. echo 1> net.bridge.bridge-nf-call-iptables = 1
2. modprobe br_netfilter
3. проверяем командой sysctl -p /etc/sysctl.conf

[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
1. echo 1 > /proc/sys/net/ipv4/ip_forward

Производим запуск:

```
kubeadm init --pod-network-cidr=10.19.0.0/16
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Установка сетевого плагина weave

```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

Результаты:

```
kubectl get pods -A
NAMESPACE     NAME                            READY   STATUS    RESTARTS      AGE
kube-system   coredns-565d847f94-6jh95        1/1     Running   0             3m14s
kube-system   coredns-565d847f94-nzjn5        1/1     Running   0             3m14s
kube-system   etcd-astra                      1/1     Running   2             3m29s
kube-system   kube-apiserver-astra            1/1     Running   2             3m30s
kube-system   kube-controller-manager-astra   1/1     Running   2             3m28s
kube-system   kube-proxy-w9h4j                1/1     Running   0             3m14s
kube-system   kube-scheduler-astra            1/1     Running   2             3m31s
kube-system   weave-net-s5c8q                 2/2     Running   1 (21s ago)   31s
```
