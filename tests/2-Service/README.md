### C'est quoi un service ?

Une abstraction qui expose un pod aux autres services.
Chaque pod a un service associé du fait du caractère éphémère d'un pod. Le service lui est fixe. On appelle/joint un pod via son service.

Il en existe 3 types :
- ClusterIP : uniquement interne au cluster
- NodePort : expose un port fixe
- LoadBalancer : expose le pod via l'adresse IP externe

### Bonnes pratiques

### Je teste quoi ?

L'exposition à travers les différents services du déploiement précédent (sauf LoadBalancer du fait d'absence de resource le permettant).
```
$ k get pods -n demo -o wide
NAME                      READY   STATUS    RESTARTS   AGE     IP           NODE        NOMINATED NODE   READINESS GATES
my-app-5775d6cfff-f42tf   1/1     Running   0          2m32s   10.42.0.38   master-01   <none>           <none>
my-app-5775d6cfff-ldgvk   1/1     Running   0          2m40s   10.42.0.37   master-01   <none>           <none>
my-app-5775d6cfff-qc4vl   1/1     Running   0          2m46s   10.42.0.36   master-01   <none>           <none>

$ k apply -f tests/2-Service/svc-test.yaml 

$ k get svc -n demo
k get svc -n demo
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
svc-clusterip   ClusterIP   10.43.2.44      <none>        80/TCP         63s
svc-nodeport    NodePort    10.43.204.132   <none>        80:31888/TCP   63s

$ k describe svc -n demo my-app 
Name:                     svc-nodeport
Namespace:                demo
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.204.132
IPs:                      10.43.204.132
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31888/TCP
Endpoints:                10.42.0.38:80,10.42.0.36:80,10.42.0.37:80
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
Events:                   <none>

$ k describe svc -n demo svc-clusterip 
Name:                     svc-clusterip
Namespace:                demo
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.2.44
IPs:                      10.43.2.44
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                10.42.0.37:80,10.42.0.38:80,10.42.0.36:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```
On constate bien les que endpoints sont bien les IPs des pods du déploiement.

On teste la haute dispo pour valider que chaque pod répond (via son hostname dans le index.html)
```
# test du service NodePort
$ for i in $(seq 1 6); do curl -s http://localhost:30978; done
Hello from my-app-5775d6cfff-f42tf with env production
Hello from my-app-5775d6cfff-qc4vl with env production
Hello from my-app-5775d6cfff-ldgvk with env production
Hello from my-app-5775d6cfff-f42tf with env production
Hello from my-app-5775d6cfff-f42tf with env production
Hello from my-app-5775d6cfff-f42tf with env production

# test du service ClusterIP
$ for i in $(seq 1 6); do curl -s http://10.43.2.44:80; done
Hello from my-app-5775d6cfff-ldgvk with env production
Hello from my-app-5775d6cfff-ldgvk with env production
Hello from my-app-5775d6cfff-qc4vl with env production
Hello from my-app-5775d6cfff-f42tf with env production
Hello from my-app-5775d6cfff-qc4vl with env production
Hello from my-app-5775d6cfff-qc4vl with env production
```