# Pourquoi K3s ?

Parce que c'est léger, facile de mise en oeuvre, facile à supprimer si besoin de repartir de zéro.
Donc parfait pour expérimenter.

# Installation

> Les 3 noeuds sont sur Debian 13

Je génère au préalable de l'installation le fichier de config qui sera lu par le batch d'install
```
$ sudo mkdir -p /etc/rancher/k3s
# l'ingress traefik et le loadbalancer servicelb seront installés plus tard, manuellement
$ sudo vi /etc/rancher/k3s/config.yaml
disable:
  - traefik
  - servicelb
```
L'installation se fait ensuite facilement et très rapidement via :
``curl -sfL https://get.k3s.io | sh -``
```
$ sudo kubectl get nodes
NAME        STATUS   ROLES           AGE   VERSION
master-01   Ready    control-plane   6s    v1.36.5+k3s1
```
Je copie le kubeconfig sur mon profil afin qu'il soit lisible sans utiliser sudo
```
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```
Je crée l'alias k pour kubectl, exporte l'emplacement du kubeconfig et ajoute l'autocomplétion :

```
$ echo 'alias k=kubectl' >> ~/.bashrc
$ echo "export KUBECONFIG=~/.kube/config" >> ~/.bashrc
$ echo 'source <(kubectl completion bash)' >> ~/.bashrc
$ echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc
$ source ~/.bashrc
$ k get nodes
NAME        STATUS   ROLES           AGE   VERSION
master-01   Ready    control-plane   86m   v1.36.5+k3s1
```

On valide le fonctionnement

```
# Création d'un namespace demo et d'un serveur nginx
$ k create ns demo
$ k run nginx --image nginx -n demo
$ k get pods -n demo
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          15s

# On port-forward sur le port 8080
$ k port-forward pods/nginx 8080:80 -n demo

# Depuis un second shell
$ curl -I localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.31.7
Date: Wed, 21 Oct 2026 09:18:04 GMT
Content-Type: text/html
Content-Length: 896
Last-Modified: Thu, 01 Oct 2026 17:19:27 GMT
Connection: keep-alive
ETag: "6abe961f-380"
Accept-Ranges: bytes

# On détruit le ns et ce qu'il y a dedans
$ k delete ns demo
namespace "demo" deleted
$ k get pods -n demo -o wide
No resources found in demo namespace.