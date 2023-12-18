# Kubernetes-by-k3s

[K3s](https://k3s.io/) - это полностью совместимый c Kubernetes облегчённый дистрибутив.

* Сертифицирован [CNCF](https://www.cncf.io/)(Сloud Native Computing Foundation). Это означает, что он прошел 
тесты на соответствие, используемые CNCF для сертификации дистрибутивов Kubernetes.
* Является [проектом песочницы](https://www.cncf.io/projects/k3s/) CNCF.

[Документация](https://rancher.com/docs/k3s/latest/en/).

## Firewall

Рекомендуется отключить Firewall, так как зачастую в локальных сетях это ложится на плечи отдельного роутера или в вашей сети есть аппаратный вариант Firewall

### Ubuntu / Debian
```shell
ufw disable
```
### RHEL / Centos / Fedora
```shell
systemctl disable firewalld --now
```

Но если вы решили не отключать его, то нужно настроить следующий список портов

### Ubuntu / Debian
```shell
ufw allow 6443/tcp #apiserver
ufw allow from 10.42.0.0/16 to any #pods
ufw allow from 10.43.0.0/16 to any #services
```

### RHEL / Centos / Fedora
```shell
firewall-cmd --permanent --add-port=6443/tcp #apiserver
firewall-cmd --permanent --zone=trusted --add-source=10.42.0.0/16 #pods
firewall-cmd --permanent --zone=trusted --add-source=10.43.0.0/16 #services
firewall-cmd --reload
```

За дополнительными настройками Firewall при различных изменениях CIDR или уникальных настроек хост машины, лучше обьратиться на сайт с официальной [документацией](https://docs.k3s.io/installation/requirements?os=rhel)


## Установка

### Первый мастер.

```shell
curl -sfL https://get.k3s.io | K3S_TOKEN='<your-secret-token>' sh -s - server --cluster-init 
```

Можно с дополнительными параметрами:

```shell
curl -sfL https://get.k3s.io | K3S_TOKEN='<your-secret-token>' sh -s - server --cluster-init \
     --flannel-iface "ens33" \
     --cluster-cidr "10.223.0.0/18"  --service-cidr "10.224.0.0/18" --cluster-dns "10.223.0.10" \
     --default-local-storage-path "/var/k3s/storage" \
     --data-dir "/var/k3s/data"
```

```shell
kubectl get nodes
```

### Дополнительные мастера.

Устанавливаем остальные мастера. Общее количество мастеров должно быть нечётным.

```shell
curl -sfL https://get.k3s.io | K3S_TOKEN='<your-secret-token>' sh -s - server --server https://k3sm1.kryukov.local:6443
```

Если с дополнительными параметрами:

```shell
curl -sfL https://get.k3s.io | K3S_TOKEN='<your-secret-token>' sh -s - server --server https://k3sm1.kryukov.local:6443 \
     --flannel-iface "ens33" \
     --cluster-cidr "10.223.0.0/18"  --service-cidr "10.224.0.0/18" --cluster-dns "10.223.0.10" \
     --default-local-storage-path "/var/k3s/storage" \
     --data-dir "/var/k3s/data"
```

### Worker ноды.

На мастер ноде получаем токен.

```shell
cat /var/lib/rancher/k3s/server/node-token

```

Подставляем его в переменную и запускаем установку.

```shell
curl -sfL https://get.k3s.io | K3S_URL="https://k3sm1.kryukov.local:6443" \
      K3S_TOKEN='K1079038baf34fd74abdb5f5cbc38018cf30b86500f7b28e0934103f23d9cfb8d89::server:l%TH]c4VvCT<Xj{' \
      sh -
```

Если необходимо использовать дополнительные параметры, определяйте их в переменной ```INSTALL_K3S_EXEC```:

```shell
curl -sfL https://get.k3s.io | K3S_URL="https://k3sm1.kryukov.local:6443" \
      K3S_TOKEN='K1079038baf34fd74abdb5f5cbc38018cf30b86500f7b28e0934103f23d9cfb8d89::server:l%TH]c4VvCT<Xj{' \
      INSTALL_K3S_EXEC="--flannel-iface ens33 --data-dir /var/k3s/data" \
      sh -
```