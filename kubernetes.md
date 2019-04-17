# Kubernetes

## Prérequis

### Mettre à jour l'environement

```shell
$ sudo su
# apt-get update
# yum install nano
```

### Activer le SSH pour se connecter à distance

```shell
# systemctl enable sshd
# systemctl start sshd
```

### Désactiver le swap

Passer le `swap` en `off`

```shell
# swapoff -a
```

Supprimer le `swap` au démarage de la machine

```shell
# nano /etc/fstab
```

Ajouter un `#` devant la ligne `swap`

### Installer Docker

Installer la dernière version de docker avec 

```shell
# yum install docker
# systemctl enable docker
```

## Installer Kubernetes

### Ajouter le repository des sources pour kubernetes

Pour plus d'informations voir sur le site du [Kubernetes](https://kubernetes.io/fr/docs/setup/independent/install-kubeadm/)

```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
```

### Finalisation du paramétrage

```bash
# Mettre SELinux en mode permissif (le désactiver efficacement)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# Installation de kubernetes
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# Activation du service
systemctl enable --now kubelet
```

## Démarage de Kubernetes

Exécuter les commandes suivantes en tant qu'utilisateur

Relancer une mise à jour

```
$ yum update
```

### Création d'un cluster master

```
$ sudo kubeadm init --apiserver-advertise-address=<host ip> --pod-network-cidr=192.168.0.0/16
```

Et attendre que le programme télécharge les images

Récupérer les informations qui ont été affiché par le programme à la fin de l'installation

Les informations sont affichés du type 

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.144.132:6443 --token 344ijo.ua0m41srf0abts47 \
    --discovery-token-ca-cert-hash sha256:aa4e2e70c82f30acb1a82d56ef80b4a6d8977161826770638a618d99ef47c4bf
```

Exécuter les commandes comme défini dans le texte

Créer le pod dashboard kubernetes 

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
```

Vérifier que le pod à bien été créé

```
kubectl get pods -o wide --all-namespaces
```

Ici on peut voir que le dashboard est en 0/1. 

Lancer le proxy dans une nouvelle fenêtre du terminal

```
kubectl proxy
```

Et se rendre sur l'url `http://localhost:8001/` pour vérifier que l'api répond.






Supprimer un pod
```
kubectl delete pod -n=kube-system <pod name>
```

Ressources 
- `https://www.youtube.com/watch?v=UWg3ORRRF60&t=286s`
- `https://github.com/kubernetes/dashboard/wiki/Accessing-Dashboard---1.7.X-and-above#kubectl-proxy`
- `https://www.dadall.info/article659/demarrer-notre-cluster-kubernetes`