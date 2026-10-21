### C'est quoi un namespace ?

C'est la ressource k8s qui cloisonne logiquement les ressources. Plusieurs logiques sont applicables :
- un namespace par environnement (dev, stage, prod)
- un namespace par projet (app1, app2, app3)
- un namespace par service (dev, ops, sre) ...

L'isolation apportée :
- les ressources de namespace différent peuvent portées le même nom
- les Secret/ConfigMap ne sont pas accessibles d'un Namespace à un autre
- un Service ne peut accéder qu'aux pods du même namespace

Ce n'est pour autant pas une sécurité, il faut :
- RBAC : pour restreindre l'accès aux namespaces aux utilisateurs
- NetworkPolicy : pour restreindre les communications entre pods de différents namespaces

> **Depuis un pod dans dev, on peut joindre un service dans prod via son adresse complète**
>
> ``kubectl exec -it mon-pod -n dev -- curl http://mon-service.prod.svc.cluster.local``

### Bonnes pratiques

Mettre en place RBAC/NetworkPolicy

Si on travaille principalement sur un Namespace on peut le fixer au contexte, ce qui évite de devoir le nommer à chaque commande :

``sudo kubectl config set-context default ---namespace dev``

### Je teste quoi ?

L'utilisation de Namespace pour gérer une même application au sein de 2 environnements distincts (dev et prod)

```
$ k apply -f tests/3-Namespace/ns-test.yaml 
$ k get pods -l app=web-app --all-namespaces 
NAMESPACE   NAME                       READY   STATUS    RESTARTS   AGE
dev         web-app-67759c4654-7sp54   1/1     Running   0          2m15s
dev         web-app-67759c4654-mwnh9   1/1     Running   0          2m15s
prod        web-app-5df965798b-5jpzn   1/1     Running   0          2m15s
prod        web-app-5df965798b-7ldtb   1/1     Running   0          2m15s
prod        web-app-5df965798b-tn5wc   1/1     Running   0          2m15s

$ k get pods -l environment=dev --all-namespaces
NAMESPACE   NAME                       READY   STATUS    RESTARTS   AGE
dev         web-app-67759c4654-7sp54   1/1     Running   0          4m8s
dev         web-app-67759c4654-mwnh9   1/1     Running   0          4m8s

$ k get pods -l app=web-app,environment=dev -n dev 
NAME                       READY   STATUS    RESTARTS   AGE
web-app-67759c4654-7sp54   1/1     Running   0          6m50s
web-app-67759c4654-mwnh9   1/1     Running   0          6m50s

$ sudo kubectl config set-context default --namespace dev
Context "default" modified.
$ k get pods -l app=web-app,environment=dev
NAME                       READY   STATUS    RESTARTS   AGE
web-app-67759c4654-7sp54   1/1     Running   0          10m
web-app-67759c4654-mwnh9   1/1     Running   0          10m