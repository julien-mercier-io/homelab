# Bonnes pratiques

- Toujours fixer tag + digest des images.
    - Le tag pour la partie lisible par un humain,
    - le digest pour la partie immuable/reproductible
- Définir les probes readiness/liveness
    - valeurs à définir en fonction de l'app
    - si liveness échoue : pod redémarré
    - si readiness échoue : pod retiré du service
- Définir les resources request/limit
    - Burstable si limits > requests
    - Guaranteed si limits = requests

# Je teste quoi ?

Un déploiement classique, une variable d'environnement, un service associé, un scale (up and down)

1er déploiement, on vérifie

```
$ k create ns demo
$ k apply -f tests/Deployment/deploy-test.yaml
$ k get deployments.apps -n demo
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
my-app   3/3     3            3           29m
$ k get pods -n demo 
NAME                      READY   STATUS    RESTARTS   AGE
my-app-7b8d87748d-6lsmv   1/1     Running   0          9m10s
my-app-7b8d87748d-76m9d   1/1     Running   0          9m11s
my-app-7b8d87748d-vsz8v   1/1     Running   0          9m10s
$ k get svc -n demo
NAME     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
my-app   NodePort   10.43.255.200   <none>        80:31447/TCP   12m
```
On teste la haute dispo pour valider que chaque pod répond (via son hostname dans le index.html)
```
$ k get svc my-app -n demo -o jsonpath='{.spec.ports[0].nodePort}'
31447
$ for i in $(seq 1 10); do curl -s http://localhost:31447; echo; done
<h1>Hello from my-app-7b8d87748d-6lsmv with env production</h1>
<h1>Hello from my-app-7b8d87748d-vsz8v with env production</h1>
<h1>Hello from my-app-7b8d87748d-6lsmv with env production</h1>
<h1>Hello from my-app-7b8d87748d-76m9d with env production</h1>
<h1>Hello from my-app-7b8d87748d-76m9d with env production</h1>
<h1>Hello from my-app-7b8d87748d-6lsmv with env production</h1>
<h1>Hello from my-app-7b8d87748d-76m9d with env production</h1>
<h1>Hello from my-app-7b8d87748d-76m9d with env production</h1>
<h1>Hello from my-app-7b8d87748d-vsz8v with env production</h1>
<h1>Hello from my-app-7b8d87748d-6lsmv with env production</h1>
```
On scale up and down
```
$ k scale deployment my-app -n demo --replicas 5
deployment.apps/my-app scaled
$ k get deployments.apps -n demo 
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
my-app   5/5     5            5           42m

$ k scale -n demo deployment my-app --replicas 3
deployment.apps/my-app scaled
$ k get deployments.apps -n demo 
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
my-app   3/3     3            3           42m