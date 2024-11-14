use https://github.com/ubc/mediawiki-docker.git or https://www.mediawiki.org/wiki/MediaWiki-Docker
and need to see https://gist.github.com/mauriballes/3525c4057b43284dbd2facfbb55ad000
and Postgres stolon https://github.com/sorintlab/stolon
______________________________________________________________________________________________________________
INTALL K8S

Инсталяция k8s по https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

0. https://kubernetes.io/releases/version-skew-policy/#supported-versions <- информация по совместимости версий

______________________________________________________________________________________________________________________________________

1. https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/
   https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/




	ip link - проверить уникальность mac-адреса
	sudo cat /sys/class/dmi/id/product_uuid - тоже проверить уникальность

	https://kubernetes.io/docs/reference/networking/ports-and-protocols/ - проверить открытые порты:




	Control plane

	Protocol	Direction	Port Range	Purpose			Used By
	TCP		Inbound		6443		Kubernetes API server	All
	TCP		Inbound		2379-2380	etcd server client API	kube-apiserver, etcd
	TCP		Inbound		10250		Kubelet API		Self, Control plane
	TCP		Inbound		10259		kube-scheduler		Self
	TCP		Inbound		10257		kube-controller-manager	Self



	Worker node(s)

	Protocol	Direction	Port Range	Purpose			Used By
	TCP		Inbound		10250		Kubelet API		Self, Control plane
	TCP		Inbound		10256		kube-proxy		Self, Load balancers
	TCP		Inbound		30000-32767	NodePort Services†	All





	sudo swapoff -a


______________________________________________________________________________________________________________________________________

master:

iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

iptables -A OUTPUT -o enp3s0 -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A OUTPUT -p icmp -j ACCEPT

iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 22 -j ACCEPT

iptables -P INPUT ACCEPT

iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 6443 -s 10.120.0.202/32 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 6443 -s 10.120.0.185/32 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 6443 -s 10.120.0.142/32 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --match multiport --dports 2379:2380 -s 10.120.0.202/32 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --match multiport --dports 2379:2380 -s 10.120.0.185/32 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --match multiport --dports 2379:2380 -s 10.120.0.142/32 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10250 -s 10.120.0.202 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10250 -s 10.120.0.185 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10250 -s 10.120.0.142 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10259 -s 10.120.0.202 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10259 -s 10.120.0.185 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10259 -s 10.120.0.142 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10257 -s 10.120.0.202 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10257 -s 10.120.0.185 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10257 -s 10.120.0.142 -j ACCEPT


______________________________________________________________________________________________________________________________________

workers:

iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

iptables -A OUTPUT -o enp3s0 -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A OUTPUT -p icmp -j ACCEPT

iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 22 -j ACCEPT

iptables -P INPUT ACCEPT

iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --match multiport --dports 30000:32767 -s 10.120.0.190 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10250 -s 10.120.0.190 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10256 -s 10.120.0.190 -j ACCEPT

iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --match multiport --dports 30000:32767 -s 10.120.0.254 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10250 -s 10.120.0.254 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10256 -s 10.120.0.254 -j ACCEPT

iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --match multiport --dports 30000:32767 -s 10.120.0.213 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10250 -s 10.120.0.213 -j ACCEPT
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m tcp --dport 10256 -s 10.120.0.213 -j ACCEPT

______________________________________________________________________________________________________________________________________


1. Installing a container runtime


1.2 Container Runtimes - https://kubernetes.io/docs/setup/production-environment/container-runtimes/

containerd -> getting started with containerd (https://github.com/containerd/containerd/blob/main/docs/getting-started.md)

1.2.1 Install containerd AND you will have to install runc and CNI plugins from their official sites too

1.2.1.1 CONTAINERD

https://github.com/containerd/containerd/releases


1.2.1.1.1

Download the containerd-<VERSION>-<OS>-<ARCH>.tar.gz archive from https://github.com/containerd/containerd/releases , verify its sha256sum, and extract it under /usr/local

	$ tar Cxzvf /usr/local containerd-1.6.2-linux-amd64.tar.gz

hint: для версии containerd 2.0 добавлен пункт Configure otel from env instead of config.toml


1.2.1.2
Typically, you will have to install runc and CNI plugins from their official sites too

https://github.com/opencontainers/runc/releases

https://github.com/containernetworking/plugins/releases
________________________

________________________

1.2.2 Install and configure prerequisites


To manually enable IPv4 packet forwarding:

____________

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system


sysctl net.ipv4.ip_forward

____________



1.2.3 cgroup drivers

если по-умолчанию в качестве системы инициализации для дистрибутива Linux используется systemd - то необходим драйвер systemd cgroup

пишется в https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/

____________

#это настраивает systemd как драйвер cgroup для kubelet

apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
...
cgroupDriver: systemd





apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: "192.168.0.8"
port: 20250
serializeImagePulls: false
evictionHard:
    memory.available:  "100Mi"
    nodefs.available:  "10%"
    nodefs.inodesFree: "5%"
    imagefs.available: "15%"


In this example, the kubelet is configured with the following settings:

address: The kubelet will serve on IP address 192.168.0.8.

port: The kubelet will serve on port 20250.

serializeImagePulls: Image pulls will be done in parallel.

evictionHard: The kubelet will evict Pods under one of the following conditions:

When the node's available memory drops below 100MiB.
When the node's main filesystem's available space is less than 10%.
When the image filesystem's available space is less than 15%.
When more than 95% of the node's main filesystem's inodes are in use.


____________


!!! при этом:

Начиная с версии 1.22 и более поздних версий, при создании кластера с помощью kubeadm, если пользователь не задает cgroupDriverполе в разделе KubeletConfiguration, kubeadm по умолчанию устанавливает значение systemd.


!!! Если вы настраиваете systemdкак драйвер cgroup для kubelet, вы также должны настроить systemdкак драйвер cgroup для среды выполнения контейнера. Инструкции см. в документации для среды выполнения контейнера. Например:

контейнерd -> https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd
CRI-O      -> https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o



systemdкак драйвер cgroup для среды выполнения контейнера:

/etc/containerd/config.toml -> section runc

Настройка systemdдрайвера cgroup
____________

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
____________


В Kubernetes 1.31 при включенном KubeletCgroupDriverFromCRI шлюзе функций и среде выполнения контейнера, поддерживающей RuntimeConfigCRI RPC, kubelet автоматически обнаруживает соответствующий драйвер cgroup из среды выполнения и игнорирует настройки cgroupDriverв конфигурации kubelet.


1.3.1 https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/


Start a kubelet process configured via the config file

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/kubelet-integration/



2. https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

