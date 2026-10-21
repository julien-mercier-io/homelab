### C'est quoi un deployment ?

C'est la ressource k8s qui déploie une application sur le cluster par l'intermédiaire d'un replicaset.
Deployment > ReplicaSet > Pod(s)

### Bonnes pratiques

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

### Je teste quoi ?

Un déploiement classique, une variable d'environnement, un scale (up and down)

1er déploiement, on vérifie

```
$ k create ns demo
$ k apply -f tests/1-Deployment/deploy-test.yaml
$ k get deployments.apps -n demo
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
my-app   3/3     3            3           29m
$ k get pods -n demo -o wide
NAME                      READY   STATUS        RESTARTS      AGE     IP           NODE        NOMINATED NODE   READINESS GATES
my-app-6dd9cc9895-9n4vz   1/1     Running       0             42s     10.42.0.34   master-01   <none>           <none>
my-app-6dd9cc9895-cn2q4   1/1     Running       0             49s     10.42.0.33   master-01   <none>           <none>
my-app-6dd9cc9895-lmvv9   1/1     Running       0             35s     10.42.0.35   master-01   <none>           <none>
$ curl 10.42.0.34
Hello from my-app-6dd9cc9895-9n4vz with env production
$ curl 10.42.0.35
Hello from my-app-6dd9cc9895-lmvv9 with env production
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