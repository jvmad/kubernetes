---------------------------------------------------------------------------------------------------------------
# Formation Kubernetes
---------------------------------------------------------------------------------------------------------------


---------------------------------------------------------------------------------------------------------------
## Security Cluster
---------------------------------------------------------------------------------------------------------------
https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/
https://kubernetes.io/docs/tasks/administer-cluster/highly-available-master/




---------------------------------------------------------------------------------------------------------------
## Les Nodes
---------------------------------------------------------------------------------------------------------------
Exemple pour la création d'un objet k8s de type node:
```yaml
apiVersion: v1
kind: node
metadata:
   name: < ip address of the node>
   labels:
      name: <lable name>
```
Information: Les noeuds Kubernetes peuvent être programmés sur Capacité. Les pods peuvent utiliser toute la capacité disponible sur un nœud par défaut : https://kubernetes.io/docs/tasks/administer-cluster/reserve-compute-resources/




---------------------------------------------------------------------------------------------------------------
## Kubectl
---------------------------------------------------------------------------------------------------------------
### Installation Kubectl:

1/ Télécharger le binaire:
```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
```

2/ Rendre le binaire kubectl exécutable:
```bash
 chmod +x ./kubectl
```

3/ Déplacez le binaire dans le PATH:
```bash
 sudo mv ./kubectl /usr/local/bin/kubectl
```

4/ Activer de l'auto-complétion en exécutant :
```bash
$ source <(kubectl completion bash)
```

Pour ajouter l'autocomplétion à votre profil:
```bash
 echo "source <(kubectl completion bash)" >> ~/.bashrc 
```

5/ vérifier la version et l'aide Kubectl:
```bash
$ kubectl version
$ kubectl -h
```



### Configurer Kubectl:
Il faut configurer kubectl en local pour pouvoir interagir avec le cluster Kubernetes (IP de l'apiserver, certificats client, Token,informations d'identification utilisateur. Spécifier une option qui existe déjà fusionnera de nouveaux champs avec les valeurs existantes pour ces champs. Par défaut, la configuration de kubectl est située à ~/.kube/config.

Pour modifier le fichier kubeconfig:
```bash
$ kubectl config -h
$ kubectl <command> --help
$ kubectl config –-kubeconfig <String of File name>
```
Pour obtenir une liste d’options globales:
```bash
$ kubectl options
```

1/ Définir une entrée de cluster dans Kubeconfig :
kubectl config set-cluster NAME [--server=server] [--certificate-authority=...] [--insecure-skip-tls-verify=true] [options]
*--insecure-skip-tls-verify=true(Désactive la vérification de certification)

```bash
$ kubectl config get-clusters
$ kubectl config set-cluster $CLUSTER_NAME --certificate-authority=ca.pem --embed-certs=true --server=https://$MASTER_IP
```
Pour supprimer le cluster:
```bash
$ kubectl config delete-cluster <$CLUSTER_NAME>
```

2/ Définir les credentials (entrée utilisateur):
```bash
$ kubectl config set-credentials -h
$ kubectl config set-credentials $USER --client-certificate=$CLI_CERT --client-key=admin-key.pem --embed-certs=true --token=$TOKEN
ou
$ kubectl config set-credentials default-admin --certificateauthority = ${CA_CERT} --client-key = ${ADMIN_KEY} --clientcertificate = ${ADMIN_CERT}
```

3/ Définissez le context par défaut dans Entrypoint k8s:
```bash
$ kubectl config get-contexts
$ kubectl config set-context $CONTEXT_NAME --cluster=$CLUSTER_NAME --user=$USER --namespace=$namespace
$ kubectl config use-context $CONTEXT_NAME
$ kubectl config current-context

Pour supprimer un contexte spécifié de kubeconfig:
$ kubectl config delete-context <Context Name>
```

3/ Vérifier la configuration :
```bash
$ kubectl config view
```

4/ Afficher les versions d'API prises en charge sur le cluster.
```bash
$ kubectl api-versions
```



### Vérification l'état du cluster:
1/ Affiche les informations du cluster. Vérifier que kubectl est correctement configuré et a accès au cluster. 
```bash
 $ kubectl cluster-info 
 *L’URL indique que kubectl est correctement configuré pour accéder au cluster (Apiserver)
 ```
Dans le cas contraire, vérifiez que celui-ci est correctement configuré:
Affiche les informations pertinentes concernant le cluster pour le débogage et le diagnostic.
```bash
 $ kubectl cluster-info dump
 $ kubectl cluster-info dump --output-directory=/path/to/cluster-state
```

2/ Vérifiez le status de chaque composant:
```bash
$ kubectl get cs
$ kubectl get all --all-namespaces
$ kubectl get pods --all-namespaces
```

5/ Obtenez des informations sur les nodes du cluster:
```bash
$ kubectl get nodes
$ kubectl describe nodes
```

6/ Exécute kubectl en mode reverse-proxy et tester le serveur API et l'authentification : 
```bash
$ kubectl proxy
#$ kubectl proxy --port=8080 &
$ curl http://localhost:8080/
```
*kubectl proxy crée un serveur proxy entre votre ordinateur et le serveur API Kubernetes. Par défaut, il n’est accessible que localement (à partir de la machine qui l’a démarré). 

-Info Dashboard-
Une fois le serveur proxy démarré, vous devriez pouvoir accéder à Dashboard à partir de votre navigateur. Pour accéder au point de terminaison HTTPS du tableau de bord, accédez à: 
"http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/"

*REMARQUE: Le tableau de bord ne doit pas être exposé publiquement à l'aide de la commande kubectl proxy car elle autorise uniquement la connexion HTTP. Pour les domaines autres que localhost et 127.0.0.1 il ne sera pas possible de se connecter. Rien ne se passera après avoir cliqué sur le bouton Sign in sur la page de connexion.
https://github.com/kubernetes/dashboard/wiki/Accessing-Dashboard---1.7.X-and-above


7/ Obtenez une liste et l'URL du reverse Proxy de l'ensemble des services demarré sur le cluster:
```bash
$ kubectl get all
$ kubectl get services --namespace=kube-system 
```

8/ Baculer un node en mode maintenance ou normal:
```bash
$ kubectl drain $NODENAME
$ kubectl uncordon $NODENAME
```
Pour vider en toute sécurité un node tout en respectant les SLO d'applicationVoir: 
https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/




### Utiliser Kubectl:
Lorsque d'une opération sur plusieurs ressources, on peut spécifier chaque ressource par type d'objet et nom ou spécifier un ou plusieurs fichiers:

1/ spécifier des ressources du même type:
```bash
$ kubectl get pod example-pod1 example-pod2
```

2/ spécifier plusieurs types de ressources individuellement: 
```bash
$ kubectl get pod/example-pod1 replicationcontroller/example-rc1
```

3/ spécifier les ressources avec un ou plusieurs fichiers: 
```bash
$ kubectl get pod -f ./pod.yaml
```

4/ Field Selectors:
Les sélecteurs de champs vous permettent de sélectionner les ressources Kubernetes en fonction de la valeur d'un ou de plusieurs champs de ressources. Les sélecteurs de champs sont essentiellement des filtres de ressources. Par défaut, aucun sélecteur / filtre n'est appliqué, ce qui signifie que toutes les ressources du type spécifié sont sélectionnées. 
metadata.name=my-service
metadata.namespace!=default
status.phase=Pending

```bash
$ kubectl get pods
$ kubectl get pods --field-selector ""
```

Vous pouvez utiliser les opérateurs = , == et != Avec des sélecteurs de champs ( = et == signifient la même chose).
```bash
$ kubectl get services --field-selector metadata.namespace!=default
```

voir: https://kubernetes.io/docs/concepts/overview/object-management-kubectl/overview/



### Migration de commandes
Migration des commandes impératives vers la configuration d'objet impérative:

1/ Exportez l'objet live dans un fichier de configuration d'objet yaml:
```bash
$ kubectl get <kind>/<name> -o yaml --export > xxx.yaml
```

2/ Modifer manuellement les champs du nouveau fichier.

3/ Pour la gestion d'objet ultérieure, utilisez replace exclusivement: 
```bash
$ kubectl replace -f xxx.yaml 
```



---------------------------------------------------------------------------------------------------------------
## Manifeste Template:
---------------------------------------------------------------------------------------------------------------
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: monpod
  namespace: namespace1
  annotations:
    description: my frontend
  labels:
    environment: "production"
    tier: "frontend"
spec:
  restartPolicy: Always
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: container1
    image: ubuntu
    imagePullPolicy: Always
    env:
    -name: envname
     value: "valenv"
    ressources:
     limits:
      memory: "200Mi"
      cpu: "1"
      ephemeral-storage: "4Gi"
     requests: 
      memory: "100Mi"
      cpu: "0.5"
      ephemeral-storage: "2Gi"
    securityContext:
     allowPrivilegeEscalation: false
    livenessProbe:
     initialDelaySeconds: 15
     timeoutSeconds: 1
     httpGet:
      path: /site
      port: 8080
      httpHeaders:
      - name: X-Custom-Header
        value: Awesome
    command: ['sh','-c','echo Hello onepoint! && sleep ]
    #args: -cpus "2"
    
    *Args:-cpus "2" dit au conteneur d'essayer d'utiliser 2 cpus. Mais le conteneur est seulement autorisé à utiliser environ 1 CPU. 
```



---------------------------------------------------------------------------------------------------------------
## Traduire un fichier Docker Compose en Kompose
---------------------------------------------------------------------------------------------------------------
Traduire un fichier Docker Compose en ressources Kubernetes (Kubernetes + Compose = Kompose)
C'est un outil de conversion (Docker Compose) pour les orchestrateurs de conteneurs (Kubernetes ou OpenShift).
http://kompose.io/

Installation kompose:
```bash
curl -L https://github.com/kubernetes/kompose/releases/download/v1.17.0/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose
```

1/ Utilisation en trois étapes :
 - Accédez au répertoire contenant votre fichier "docker-compose.yaml"
 - Exécutez "kompose up" dans le même répertoire
 - Vérifiez le déploiement des containers dans le cluster Kubernetes "kubectl get po"

2/ Alternativement: vous pouvez exécuter kompose convert pour générer un fichier à utiliser avec kubectl.
```bash
$ kompose --file docker-convert.yml convert
$ kompose --provider openshift --file docker-convert.yml convert
```

Voir: https://kubernetes.io/docs/tools/kompose/user-guide/



---------------------------------------------------------------------------------------------------------------
## Service Account
---------------------------------------------------------------------------------------------------------------
Un compte de service fournit une identité pour les processus qui 'exécutent dans un pod. Lorsque un utilisateur accéde au cluster (ex: via kubectl), il est authentifié par l'Apiserver comme un "compte utilisateur" (actuellement "admin"). Les processus dans les conteneurs à l'intérieur des Pods peuvent également contacter l'Apiserver. Lorsqu'ils le font, ils sont authentifiés en tant que "compte de service" ("default").

1/ Créer un compte de service:
Lorsqu'un pod est créé, si vous ne spécifiez pas de compte de service, le compte de service "default" lui est automatiquement affecté dans le même NameSapce. Les autorisations API d'un compte de service dépendent du "plug-in d'autorisation" et de la "stratégie" utilisée. On peut désactiver les informations d'identification de l'API automounting pour un compte de service en définissant "automountServiceAccountToken: false" sur le compte de service.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: My-serviceAccount
automountServiceAccountToken: false
```

On peut également désactiver les informations d'identification de l'API automounting pour un pod particulier:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: My-serviceAccount
  automountServiceAccountToken: false
```
* La spécification de pod a la "priorité" sur le compte de service si les deux spécifient la valeur automountServiceAccountToken.


2/ lister toutes les ressources "serviceAccount" :
Chaque NameSapce a un ServiceAccount par default appelée "default". 
```bash
  $ kubectl get serviceAccounts
```


3/ Créer un objet ServiceAccount supplémentaires:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: My-serviceAccount2
```  
```bash
$ kubectl create -f serviceaccount2.yaml
```
Pour utiliser un compte de service particulié pour un pod, définissez lui le champ "spec.serviceAccountName"sur le nom du compte de service souhaitez. 
- Le compte de service doit exister au moment de la création du module, sinon il sera rejeté.
- Vous ne pouvez pas mettre à jour le compte de service d'un pod déjà créé.


4/ Supprimer un objets ServiceAccount :
```bash
$ kubectl delete serviceaccount/My-serviceAccount2
```

5/ Créez un jeton d'API pour un ServiceAccount:
* Créer un nouveau secret manuellement.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: Secret-My-serviceAccount
  annotations:
    kubernetes.io/service-account.name: My-serviceAccount
type: kubernetes.io/service-account-token
```

```bash
$ kubectl create -f Secret-My-serviceAccount.yaml
```

6/ Confirmer que le nouveau secret contient un jeton d'API pour le compte de service "My-serviceAccount".
```bash
$ kubectl describe secrets/Secret-My-serviceAccount
```



---------------------------------------------------------------------------------------------------------------
## NAMESPACE:
---------------------------------------------------------------------------------------------------------------

1/ Lister tous les namespaces:
```bash
$ kubectl get namespaces
$ kubectl get namespaces --show-labels
```

2/ Créer un namespace:
```bash
$ kubectl create namespace tst
$ kubectl create –f namespace-tst.yaml
```
```yaml  
apiVersion: v1
kind: Namespace
metadata:
  name: tst
```

3/ Obtenir des informations sur un namespace:
```bash
$ kubectl describe namespaces tst
$ kubectl get all --namespace=tst
$ kubectl get <type> --namespace=tst 
```

4/ Définir un namespace pour un objet:
```bash
$ kubectl --namespace=tst run nginx --image=nginx (obsolete)
```

5/ Définir le namespace par defaut pour toutes les commandes kubectl:
```bash
$ kubectl config set-context $(kubectl config current-context) --namespace=tst
$ kubectl config view | grep namespace
```

6/ Supprimer un namespace :
```bash
$ kubectl delete namespace tst
```
 
 7/ voir quelles ressources Kubernetes sont et ne sont pas dans un espace de noms:
 ```bash
$ kubectl api-resources --namespaced=true
$ kubectl api-resources --namespaced=false
```
 
 
---------------------------------------------------------------------------------------------------------------
## ResourceQuota:
---------------------------------------------------------------------------------------------------------------
1/ Créer un ResourceQuota
```yaml  
 apiVersion:   v1 
  kind:   ResourceQuota 
  metadata: 
    name:   quota
  spec: 
    hard: 
      requests.cpu:   "1" 
      requests.memory:   1Gi 
      limits.cpu:   "5" 
      limits.memory:   10Gi 
```
```bash
$ kubectl create -f resourcequota.yaml --namespace=tst
```

2/ Afficher les ResourceQuota:
```bash
$ kubectl get resourcequota quota --namespace=tst --output=yaml 
$ kubectl describe namespaces tst
```

3/ Modifier le fichier yaml et remplacer la configuration:
```yaml 
apiVersion:   v1 
kind:   ResourceQuota 
metadata: 
  name:   quota
spec: 
  hard: 
    limits.cpu:   "5" 
    limits.memory:   10Gi 
```

```bash
kubectl replace -f resourcequota.yaml
```

voir : https://kubernetes.io/docs/concepts/policy/resource-quotas/



---------------------------------------------------------------------------------------------------------------
## Les Pods
---------------------------------------------------------------------------------------------------------------
Traditionnellement, les espaces de noms et les groupes de contrôle servent à isoler les conteneurs Docker s'exécutant sur le même ordinateur. Dans Kubernetes, les conteneurs s'exécutant dans un pod partagent ces «facettes de l'isolement»,cela se traduit par la capacité des conteneurs d'un pod à communiquer via localhost. De plus, les conteneurs co-localisés dans un même pod peuvent également utiliser d'autres formes de communication interprocessus (sémaphores SystemV, mémoire partagée POSIX, etc.).

Pour créer une pile réseau partagée entre les deux conteneurs, Kubernetes utilise Docker pour permettre aux conteneurs de se connecter à une interface réseau existante. Les pods contiennent également des règles de transfert permettant à nos conteneurs, via l’interface de réseau virtuel cni0, d’envoyer et de recevoir des communications réseau. Ces règles de transfert permettent de diriger ou de recevoir le trafic depuis l' interface de réseau physique d'un pod 

1/ Création d’un fichier manifest (YAML) pour un Pod à un container :
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simplepod1
  namespace: tst
spec:
  containers:
  - name: container1
    image: centos
    env:
    - name: MESSAGE
      value: "Hello Word"
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo '$(MESSAGE)'; sleep 10;done"]
    resources:  #Si absent: error from server (Forbidden) car ResourceQuota present
     limits:
      memory: "200Mi"
      cpu: "1"
     requests: 
      memory: "100Mi"
      cpu: "0.5"
```


2/ Création du Pod à partir du manifest:
```bash
$ kubectl create -f simplepod1.yaml --record --namespace=tst
(--record enregistre la commande en cours dans les annotations. Utile pour une révision ultérieure)
$ kubectl describe pod simplepod1
```
*A la différences des "Multi container pod", les "Single Container Pod" peuvent être créés avec la commande kubctl run.
```bash
$ kubectl run tomcat --image=tomcat:8.0 (obsolete)
```

3/ Afficher les informations sur les Pods en cours d’execution:
```bash
$ kubectl get pod --namespace=tst
$ kubectl get pod --namespace=tst -o wide
$ kubectl get pod simplepod1 --namespace=tst -o wide
 *(-o wide permet d'afficher le Node auquel le pod a été assigné)
 $ kubectl get pod --namespace=tst -o yaml
$ kubectl get pod simplepod1 --namespace=tst -o yaml
 *(-o yaml "--output=yaml" spécifie d’afficher la configuration complette de l'objet)
```

4/ Voir des informations détaillées sur l'histoire et le status du Pod:
```bash
$  kubectl describe pod simplepod1 --namespace=tst
```

5/ Voir le résultat de la commande exécuté dans le conteneur, affichez les journaux du pod:`
```bash
$ kubectl log simplepod1
```

6/ S'attacher à un conteneur en cours d'exécution dans un Pod.
```bash
$ kubectl attach simplepod1
$ kubectl attach simplepod1 -c container1
```

7/ Exécuter une commande dans un conteneur:
 - Récupère le résultat de l'exécution de 'date' à partir du pod "simplepod1"
```bash
$ kubectl exec simplepod1 date
```
 - Récupère le résultat de l'exécution de 'date' dans le conteneur "container1" du pod "simplepod1"
```bash
$ kubectl exec simplepod1 -c container1 date
```
 - Récupérer la stdin de "container1" à partir du simplepod1 :
 ```bash
 $ kubectl exec -it simplepod1 -- /bin/bash
 $ kubectl exec simplepod1 -c container1 -it -- bash -il
```
*Dans le shell, exécutez la commande "printenv" pour répertorier les variables d’environnement.


8/ Supprimez un Pod:
```bash
$ kubectl delete pod simplepod1 --namespace=tst
$ kubectl delete --grace-period=0 --force pod simplepod1 --namespace=tst
 *(Remplacer la valeur de grace par défaut "La valeur 0 force la suppression du Pod")
```



---------------------------------------------------------------------------------------------------------------
## Partage des espaces de noms de processus entre des conteneurs
---------------------------------------------------------------------------------------------------------------
Partage des espaces de noms de processus entre des conteneurs dans un pod:
Lorsque le partage d'espace de noms de processus est activé, les processus d'un conteneur sont visibles par tous les autres conteneurs de ce Pod.
Vous pouvez utiliser cette fonctionnalité pour configurer des conteneurs coopérants, tels qu'un conteneur sidecar de gestionnaire de journaux, ou pour résoudre les images de conteneur qui n'incluent pas les utilitaires de débogage comme un shell.
préreqquis: Une porte de fonction alpha spéciale PodShareProcessNamespace doit être définie sur true sur le système: --feature-gates=PodShareProcessNamespace=true .

Le partage de l'espace de nom de processus est activé à l'aide du champ ShareProcessNamespace: 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  shareProcessNamespace: true
  containers:
  - name: nginx
    image: nginx
  - name: shell
    image: busybox
    securityContext:
      capabilities:
        add:
        - SYS_PTRACE
    stdin: true
    tty: true
```

Créer le Pod et attacher  un shell
Vous pouvez signaler les processus dans d'autres conteneurs. 
Il est même possible d'accéder à une autre image conteneur en utilisant le lien ```/proc/$pid/root``` .
# head /proc/8/root/etc/nginx/nginx.conf


Les processus sont visibles pour les autres conteneurs du module. Cela inclut toutes les informations visibles dans /proc
Les systèmes de fichiers du conteneur sont visibles par les autres conteneurs du conteneur via le lien ```/proc/$pid/root``` .




---------------------------------------------------------------------------------------------------------------
## Commande (Entrypoint) et des arguments (Cmd)
---------------------------------------------------------------------------------------------------------------
1/ Définir une commande (Entrypoint) et des arguments (Cmd) pour un conteneur:
La commande et les arguments définis ne peuvent pas être modifiés après la création du Pod.
La commande et les arguments définis dans le fichier de configuration remplacent celles fournis par l'image.
Si vous définissez des arguments, mais ne définissez pas de commande, la commande par défaut est utilisée avec les nouveaux arguments.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
  restartPolicy: OnFailure
```

Pour voir la sortie de la commande exécutée dans le conteneur, affichez les journaux du Pod:
```
$ kubectl logs command-demo 
```

Exécuter une commande dans un shell:
```yaml
command: ["/bin/sh"]
args: ["-c", "while true; do echo hello; sleep 10;done"]
```



---------------------------------------------------------------------------------------------------------------
## LimitRange & Ressources
---------------------------------------------------------------------------------------------------------------
1/ Créer un objet LimitRange
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limitrange
spec:
  limits:
  - default:
      cpu: 3
      memory: "1000Mi"
    defaultRequest:
      cpu: 1.5
      memory: "500Mi"
    type: Container
```

2/ Créez le LimitRange dans l'espace de noms
```bash
$ kubectl create -f limitrange.yaml --namespace=tst
```

3/ Créer un simplepod3 sans spécifier de ressources:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simplepod3
  namespace: tst
spec:
  restartPolicy: Always
  containers:
  - name: container3
    image: centos
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 10;done"]
```   
```bash
$ kubectl create -f limitrange.yaml --namespace=tst
```

Vérifier les ressources attribuer pour le "pod", "limirange", "ResourceQuota" et "namespace":
```bash
$ kubectl describe po simplepod3
$ kubectl describe limitrange
$ kubectl get resourcequota quota --output=yaml 
$ kubectl describe namespaces tst
```

Si un conteneur est créé dans un namespace et qu'il ne spécifie pas ses propres valeurs pour la demande de resources, le conteneur reçoit une demande par défaut correspondant au LimiRange associé a ce namespace.



---------------------------------------------------------------------------------------------------------------
## QoS pod
---------------------------------------------------------------------------------------------------------------

1/ Assigné une classe Guaranteed, Burstable et BestEffort au Pod et afficher le résultat:
```bash
$ kubectl get pod |grep -i qosClass
```



---------------------------------------------------------------------------------------------------------------
## Variables d'environnement 
---------------------------------------------------------------------------------------------------------------

1/ Utiliser les variables d'environnement pour définir les arguments:
```yaml
env:
- name: MESSAGE
  value: "hello world"
command: ["/bin/echo"]
args: ["$(MESSAGE)"]
```
* incluez le champ "env" ou "envFrom" dans le fichier de configuration.
* Les variables d'environnement définies à l'aide du champ "env" ou "envFrom" remplacent toutes les variables d'environnement spécifiées dans l'image du conteneur.
*On peut définir un argument pour un Pod en utilisant des variables d'environnement, y compris les ConfigMaps et Secrets.



2/ Définir une variable d'environnement pour un conteneur:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: MyPod
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: MyContainer
    image: Centos
    env:
    - name: MyVar1
      value: "Hello World"
    - name: MyVar2
      value: "Hello Formation"
```

-Créer le Pod et connectez vous dessus. Dans le shell, exécutez la commande "printenv" pour lister les variables d'environnement.



3/ Utiliser les champs "Pod" comme valeurs pour les variables d'environnement:
Un Pod peut utiliser des variables d'environnement pour exposer des informations sur lui-même aux Conteneurs s'exécutant dans ce même Pod. Les variables d'environnement peuvent exposer les champs Pod et les champs Conteneur. Il existe deux façons d'exposer les champs Pod et Conteneur à un conteneur en cours d'exécution:
-Variables d'environnement
-DownwardAPIVolumeFiles
* ces deux façons d'exposer les champs Pod et Conteneur s'appellent l' API Downward .

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: MyPod
spec:
  containers:
    - name: Mycontainer
      image: centos
      command: [ "sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          printenv MY_NODE_NAME MY_POD_NAME MY_POD_NAMESPACE;
          printenv MY_POD_IP MY_POD_SERVICE_ACCOUNT;
          sleep 10;
        done;
      env:
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: MY_POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
  restartPolicy: Never
```

-Créer le Pod et afficher les journaux du conteneur:
```
$  kubectl logs dapi-envars-fieldref 
```
* Le premier élément du tableau spécifie que la variable "MY_NODE_NAME" tire sa valeur du champ "spec.nodeName" du Pod. 
* De même, les autres variables d'environnement obtiennent leurs noms des autres champs Pod.



4/ Utiliser les champs "Conteneur" comme valeurs pour les variables d'environnement:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: MyPod
spec:
  containers:
    - name: Mycontainer
      image: centos
      command: [ "sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          printenv MY_CPU_REQUEST MY_CPU_LIMIT;
          printenv MY_MEM_REQUEST MY_MEM_LIMIT;
          sleep 10;
        done;
      resources:
        requests:
          memory: "32Mi"
          cpu: "125m"
        limits:
          memory: "64Mi"
          cpu: "250m"
      env:
        - name: MY_CPU_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: requests.cpu
        - name: MY_CPU_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: limits.cpu
        - name: MY_MEM_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: requests.memory
        - name: MY_MEM_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: limits.memory
  restartPolicy: Never
```
* Le premier élément du tableau spécifie que la variable "MY_CPU_REQUEST" tire sa valeur du champ "MY_CPU_REQUEST" d'un conteneur nommé test-container . 
* De même, les autres variables d'environnement obtiennent leurs valeurs à partir des champs Container.



5/ Stocké des champs Pod dans un DownwardAPIVolumeFile:
Un Pod peut utiliser un DownwardAPIVolumeFile pour exposer des informations sur lui-même à des Conteneurs s'exécutant dans le Pod:
Un DownwardAPIVolumeFile peut exposer les champs Pod et les champs Conteneur

Stocker les champs Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-downwardapi-volume-example
  labels:
    zone: us-est-coast
    cluster: test-cluster1
    rack: rack-22
  annotations:
    build: two
    builder: john-doe
spec:
  containers:
    - name: client-container
      image: k8s.gcr.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          if [[ -e /etc/podinfo/annotations ]]; then
            echo -en '\n\n'; cat /etc/podinfo/annotations; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
          readOnly: false
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "annotations"
            fieldRef:
              fieldPath: metadata.annotations
```
vous pouvez voir que le pod a un volume /etc/podinfo et que le conteneur monte le volume dans /etc/podinfo .
Regardez le tableau des items sous downwardAPI . 
Chaque élément du tableau est un DownwardAPIVolumeFile . 
Le premier élément spécifie que la valeur du champ metadata.labels du Pod doit être stockée dans un fichier nommé labels . 
Le second élément spécifie que la valeur du champ d' annotations du Pod doit être stockée dans un fichier nommé annotations .


vous stockez les champs Conteneur dans un DownwardAPIVolumeFile:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-downwardapi-volume-example-2
spec:
  containers:
    - name: client-container
      image: k8s.gcr.io/busybox:1.24
      command: ["sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          if [[ -e /etc/podinfo/cpu_limit ]]; then
            echo -en '\n'; cat /etc/podinfo/cpu_limit; fi;
          if [[ -e /etc/podinfo/cpu_request ]]; then
            echo -en '\n'; cat /etc/podinfo/cpu_request; fi;
          if [[ -e /etc/podinfo/mem_limit ]]; then
            echo -en '\n'; cat /etc/podinfo/mem_limit; fi;
          if [[ -e /etc/podinfo/mem_request ]]; then
            echo -en '\n'; cat /etc/podinfo/mem_request; fi;
          sleep 5;
        done;
      resources:
        requests:
          memory: "32Mi"
          cpu: "125m"
        limits:
          memory: "64Mi"
          cpu: "250m"
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
          readOnly: false
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "cpu_limit"
            resourceFieldRef:
              containerName: client-container
              resource: limits.cpu
          - path: "cpu_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.cpu
          - path: "mem_limit"
            resourceFieldRef:
              containerName: client-container
              resource: limits.memory
          - path: "mem_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.memory
```
vous pouvez voir que le pod a un volume /etc/podinfo et que le conteneur monte le volume dans /etc/podinfo .
Regardez le tableau des items sous downwardAPI . 
Chaque élément du tableau est un DownwardAPIVolumeFile.
Le premier élément spécifie que dans le conteneur nommé client-container , la valeur du champ limits.cpu doit être stockée dans un fichier nommé cpu_limit .



Capacités de l'API Downward:
Les informations suivantes sont disponibles pour les conteneurs via les variables d'environnement et DownwardAPIVolumeFiles:
- Le nom du nœud
- L'adresse IP du nœud
- Le nom du Pod
- L'espace de noms du Pod
- L'adresse IP du Pod
- Le nom du compte de service du pod
- L'UID du Pod
- Limite du processeur d'un conteneur
- Demande de CPU d'un conteneur
- Limite de mémoire d'un conteneur
- Demande de mémoire d'un conteneur

En outre, les informations suivantes sont disponibles via DownwardAPIVolumeFiles.
- Les Labels du Pod
- Les Annotations du Pod




---------------------------------------------------------------------------------------------------------------
## Labels & selector
---------------------------------------------------------------------------------------------------------------
1/ Afficher les labels générées pour chaque objets (pod, namespace, limitrange, resourcequota...) :
```bash
$ kubectl get pod --show-labels --namespace=tst
$ kubectl get namespaces --show-labels --namespace=tst
$ kubectl get limitrange --show-labels --namespace=tst
$ kubectl get resourcequota --show-labels --namespace=tst
```

2/ Attribuer des labels aux objets:
```bash
$ kubectl label pod simplepod1 environment=tst tier=frontend type=pod
$ kubectl label namespace tst environment=tst type=namespace
```


3/ Vérifier la présence des labels sur les objets (pod, namespace, limitrange, resourcequota...):
```bash
$ kubectl get pod --show-labels
```

3/ Effectuer une recherche avec le selector equality-base:
```bash
$ kubectl get pods -l environment=tst,tier=frontend
```
*equality-base permet de filtrer par clé et par valeur. Les objets correspondants doivent satisfaire à tous les labels spécifiées.

4/ Effectuer une recherche avec le selector set-based:
```bash
$ kubectl get pods -l 'environment in (tst),tier in (frontend)'
```

5/ Supprimer un objet par une recherche avec le selector equality-base:
```bash
$ kubectl describe po simplepod1
$ kubectl delete pods -l environment=tst,tier=frontend
```

6/ Attacher une anotation à un Pod:
```bash
$ kubectl annotate pods simplepod3 description='my frontend'
```

7/ Recréer plusieurs Pods avec des nom, labels et anotations différentes 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simplepod1
  namespace: tst
  annotations:
    description: my frontend
  labels:
    app.kubernetes.io/name:   myapp 
    app.kubernetes.io/instance:   wordpress-myapp 
    app.kubernetes.io/version:   "5.7.21" 
    app.kubernetes.io/component:   database 
    app.kubernetes.io/part-of:   wordpress 
    app.kubernetes.io/managed-by:   helm
    environment: "tst"
    tier: "frontend"
```


---------------------------------------------------------------------------------------------------------------
## Affectation Pod
---------------------------------------------------------------------------------------------------------------

1/ répertoriez les nodes disponibles sur le cluster:
```bash
$ kubectl get nodes 
```

2/ Ajouter les labels au node:
```bash
$ kubectl label nodes node1 affect=node1
```

3/ Vérifiez que le node possède bien le nouveau label : 
```bash
$  kubectl get nodes --show-labels 
```

4/ Créer un pod "simplepod4" planifié sur le node choisi grace au sélecteur de node (node qui a un label label1=var1):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simplepod4
  namespace: tst
spec:
  restartPolicy: Never
  nodeSelector:
    affect: node1
  containers:
  - name: container4
    image: centos
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 10;done"]
```
```bash
$ kubectl create -f simplepod4.yaml
```

5/ Vérifiez que le pod est bien en cours d'exécution sur le node choisi:
```bash
$ kubectl get pods --output=wide 
```



---------------------------------------------------------------------------------------------------------------
## TAINTS:
---------------------------------------------------------------------------------------------------------------

1/ Ajoutez un taints à un node:
```bash
$ kubectl taint nodes node1 Mykey=Myvalue:NoSchedule
$ kubectl describe no node1 | grep Taints
```
*L'affectation "Mykey=Myvalue:NoSchedule" au node signifie qu'aucun pod ne pourra être planifier sur node1, à moins d'avoir une tolérance correspondante.


2/ Lancer un nouveau pod "simplepod5" sur le node1 et vérifier le status:
```bash
$ kubectl describe po  simplepod5
```

3/ Spécifiez l'une des tolérances ci-dessous dans le PodSpec du simplepod5 (Operateur: Equal ou Exist) afin que celui-ci soit capable d'être programmé sur node1 :
```yaml
spec:
  tolerations:
  - key: "Mykey"
    operator: "Equal"
    value: "Myvalue"
    effect: "NoSchedule"

ou

tolerations:
- key: "key"
  operator: "Exists"
  effect: "NoSchedule"
```
*Une tolérance "correspond" à une taint si les clés sont les mêmes et les effets sont les mêmes
*Deux cas spéciaux:
 - Une key vide avec opérateur "Exists" correspond à toutes les clés, valeurs et effets, ce qui signifie que tout sera toléré.
```yaml
tolerations:
- operator: "Exists"
```

 - Un effect vide correspond à tous les effets avec la key “Mykey” .
```yaml
tolerations:
- key: "Mykey"
  operator: "Exists"
```


2/ Supprimer l'altération sur le node:
```bash
$ kubectl taint nodes node1 Mykey:NoSchedule-
```


Voir: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/


---------------------------------------------------------------------------------------------------------------
## Les sondes (probes)
---------------------------------------------------------------------------------------------------------------
1/ Définir un pod avec une sonde d'activité qui utilise une requête EXEC:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simplepod6
  namespace: tst
spec:
  containers:
  - name: container6
    image: centos
    command: ["/bin/sh"]
    args: ["-c", "touch /tmp/test; sleep 60; rm -rf /tmp/healthy; sleep 60"]
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/test
      initialDelaySeconds: 10
      periodSeconds: 5
```
- periodSeconds: spécifie que kubelet doit effectuer une sonde d'activité (cat /tmp/tst) toutes les 5 secondes (La valeur minimale est 1).
- initialDelaySeconds: indique a kubelet qu'il doit attendre 10 secondes avant d'effectuer la première sonde. 
- command: kubelet exécute la commande "cat /tmp/test" dans le conteneur.  Si la commande réussit, elle renvoie 0

Vérifier le status du pod:
```bash
$ kubectl describe po simplepod6
$ tail -f /var/log/message | grep -i simplepod6
```
*kubelet tue le conteneur et il est soumis à sa politique de redémarrage. La sortie "kubectl get" montre que RESTARTS a été incrémenté:


 
 
Ajouter les options suivantes:
 - timeoutSeconds: Nombre de secondes après lequel la sonde arrive à expiration (Valeur minimal par défaut "1sce").
 - successThreshold: Succès consécutifs minimum pour que la sonde soit considérée comme ayant réussi après avoir échoué. La valeur minimal par défaut est 1 (doit être 1 pour la vivacité).
 - failureThreshold: Quand un pod démarre et que la sonde échoue, Kubernetes essaiera le seuil d'échec avant d'abandonner. Abandonner en cas d’analyse signifie relancer le pod. En cas de test de disponibilité, le pod sera marqué comme étant non prêt. La valeur par défaut est 3. La valeur minimale est 1.



2/ Définir un nouveau pod avec une sonde d'activité qui utilise une requête HTTPGET:
```yaml
apiVersion: v1
apiVersion: v1
kind: Pod
metadata:
  name: simplepod7
  namespace: tst
spec:
  containers:
  - name: container7
    image: nginx
    imagePullPolicy: Always
    #command: 
    #args: 
    livenessProbe:
      httpGet:
        #path: /usr/share/nginx/html/index.html
        port: 80
        #httpHeaders:
        #- name: X-Custom-Header
        #  value: Awesome
      initialDelaySeconds: 10
      periodSeconds: 5
```
Les sondes HTTP ont des champs supplémentaires qui peuvent être définis :
- host : nom d'hôte auquel se connecter, par défaut l'adresse IP du pod. 
- scheme : Schéma à utiliser pour se connecter à l'hôte (HTTP ou HTTPS). Par défaut à HTTP.
- path : Chemin d'accès sur le serveur HTTP.
- httpHeaders : en-têtes personnalisés à définir dans la requête. HTTP permet des en-têtes répétés.
- port : nom ou numéro du port auquel accéder sur le conteneur. Le nombre doit être compris entre 1 et 65535.


3/ Définir une sonde d'activité qui utilise une requête TCPSocket:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```
- Cet exemple utilise à la fois des sondes de disponibilité (Readiness) et de vivacité (Liveness). 
- kubelet envoie la première sonde de disponibilité 5 secondes après le démarrage du conteneur. 
- Il tente de se connecter au Pod sur le port 8080. Si la sonde réussit, le pod sera marqué comme prêt. 
- kubelet continuera à exécuter cette vérification toutes les 10 secondes.

- kubelet lance la première sonde de vivacité 15 secondes après le début du conteneur. 
- Tout comme la sonde de disponibilité, il tente de se connecter au conteneur goproxy sur le port 8080. 
- Si la sonde d'activité échoue, le conteneur sera redémarré.



4/ Utiliser un port nommé:
Utiliser un ContainerPort nommé pour les contrôles d'activité HTTP ou TCP:
```yaml
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
```

Voir: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/




---------------------------------------------------------------------------------------------------------------
##  volumes
---------------------------------------------------------------------------------------------------------------
###Volume emptyDir:

1/ Répertoriez les StorageClasses du cluster:
```bash
$ kubectl get storageclass
```

2/ Définir un volume emptyDir dans le Pod et le monter dans le container avec une limite.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simplepod9
  namespace: tst
spec:
  containers:
  - image: nginx
    name: container9
    volumeMounts:
    - mountPath: /myvolume
      name: monvolume
    #ressources:
      #limits:
      #memory: "200Mi"
      #ephemeral-storage: 2GiB
      #requests: 
      #ephemeral-storage: 1GiB
  volumes:
  - name: monvolume
    emptyDir: {}
```
   
3/ Créer le Pod puis dans un autre terminal, lancer un shell sur le conteneur:
```bash
$ kubectl exec -it simplepod9 -- /bin/bash
```

4/ Dans le terminal d'origine, surveillez les modifications apportées
$ find / -name myvolume

### PV & PVC:
1/ Créer un "PersistentVolume"
```bash
$  mkdir /mnt/data 
```

2/ Dans le répertoire /mnt/data , créez un fichier index.html :
```bash
$ echo 'test volume persistant' > /mnt/data/index.html
```

3/ Créer un objet PersistentVolume à partir d'un nouveau manifest :
```yaml  
	kind: PersistentVolume
	apiVersion: v1
	metadata:
	  name: mypersvol
	  labels:
	    type: local
	spec:
	  storageClassName: manual
	  capacity:
	    storage: 10Gi
	  accessModes:
	    - ReadWriteOnce
	  hostPath:
	    path: "/mnt/data"
```
*("StorageClass"  sur manual pour un PersistentVolume est utilisé pour lier les requêtes PersistentVolumeClaim à ce PersistentVolume)
```bash
$ kubectl create –f pv.yaml
```

4/ Afficher des informations (status) sur le PersistentVolume:
```bash
$ kubectl get pv task-pv-volume
$ kubectl describe pv task-pv-volume
```

5/ Crée un PersistentVolumeClaim, qui sera automatiquement lié au PersistentVolume et qui demande un volume d'au moins trois gibibytes pouvant fournir un accès en r/w pour au moins un node:
```yaml
	kind: PersistentVolumeClaim
	apiVersion: v1
	metadata:
	  name: mypersvolclaim
	spec:
	  storageClassName: manual
	  accessModes:
	    - ReadWriteOnce
	  resources:
	    requests:
	      storage: 3Gi
```
```bash
$ kubectl create –f pvc.yaml
$ kubectl get pvc
```


6/ Regardez à nouveau le PersistentVolume et vérifier que la sortie montre que PersistentVolumeClaim est lié à votre PersistantVolume:
```bash
$  kubectl get pv task-pv-volume
$ kubectl describe pv pv0001
```

7/ Créer un pod qui utilise PersistentVolumeClaim comme volume de stockage.
```yaml
...
	spec:
	  volumes:
	  - name: volume1
	    persistentVolumeClaim:
	    claimName: mypersvolclaim
	  containers:
	   ...
	    volumeMounts:
	    - mountPath: "/var/www/monsite"
	      name: volume1
```
Dans le code ci-dessus, nous avons défini -
∙ volumeMounts: → Il s'agit du chemin dans le conteneur sur lequel le montage aura lieu.
∙ Volume: → Cette définition définit la définition de volume que nous allons réclamer.
∙ persistentVolumeClaim: → Sous cela, nous définissons le nom du volume que nous allons utiliser dans le module défini.


8/ Obtenez un shell sur le conteneur et vérifiez le montage:
```bash
$  kubectl exec -it task-pv-pod -- /bin/bash 
$  kubectl get pod task-pv-pod 
```



### Projected Volume

1/ Créer un pod pour utiliser un "Projected Volume" et pour monter les secrets (ci-dessous) dans un même répertoire partagé:
```yaml
	spec:
	  containers:
	  ...
	    volumeMounts:
	    - name: all-in-one
	      mountPath: "/projected-volume"
	      readOnly: true
	  volumes:
	  - name: all-in-one
	    projected:
	      sources:
	      - secret:
		  name: user
	      - secret:
		  name: pass
```

2/ Observez les modifications apportées au pod:
```bash
$  kubectl get --watch pod test-projected-volume 
```

3/ Dans un autre terminal vérifiez que le répertoire contient vos sources projetées
```bash
$ kubectl exec -it test-projected-volume -- /bin/sh 
$ ls /projected-volume/ 
```



---------------------------------------------------------------------------------------------------------------
## Contrôle d'accès:
---------------------------------------------------------------------------------------------------------------
Le stockage configuré avec un ID de groupe (GID) permet d'écrire uniquement par Pods en utilisant le même GID. Les GID non concordants ou manquants provoquent des erreurs d'autorisation refusées. Pour réduire le besoin de coordination avec les utilisateurs, un administrateur peut annoter un PersistentVolume avec un GID. Ensuite, le GID est automatiquement ajouté à tout Pod qui utilise le PersistentVolume.

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv1
  annotations:
    pv.beta.kubernetes.io/gid: "1234"
```    
Quand un Pod consomme un PersistentVolume qui a une annotation GID, le GID annoté est appliqué à tous les Conteneurs dans le Pod de la même manière que les GID spécifiés dans le contexte de sécurité du Pod.




---------------------------------------------------------------------------------------------------------------
## Les secrets
---------------------------------------------------------------------------------------------------------------

### Créer des secrets:

Il existe de nombreuses façons de créer des secrets dans Kubernetes.

1/ méthode 1: Création à partir de fichiers txt
-Créer des fichiers contenant le nom d'utilisateur et mot de passe:
```bash
$ echo -n "admin" > ./username.txt
$ echo -n "azerty" > ./password.txt
ou
echo -n '123456dfg45' | base64 >> ./password.txt
```
- Emballez ces fichiers dans des secrets:
```bash
$ kubectl create secret generic user --from-file=./username.txt
$ kubectl create secret generic pass --from-file=./password.txt
```

2/ mMéthode 2: Créer à partir d'une commande k8s
```bash
$ kubectl create secret generic monsecret --from-literal=username='my-app' --from-literal=password='39528$vdg7Jb'
```

3/ méthode 3: Créer à partir d'un fichier yaml
-Créer l'object secret à partir du fichier yaml:
```yaml
	apiVersion: v1
	kind: Secret
	metadata:
	  name: monsecret
	data:
	  username: admin
	  password: azerty
```

```bash
$ kubectl create –f Secret.yaml
```

4/ Afficher des informations sur le secret:
```bash
$  kubectl get secret  
$  kubectl describe secret monsecret -o yaml
```

### Utiliser des secrets
Une fois que nous avons créé les secrets, il peut être consommé dans un pod ou un contrôleur en tant que: 
 - Variable d'environnement
 - volume

5/ Créer un pod qui accède aux secret via un volume (tous les fichiers créés sur montage secret auront l'autorisation 0400):
```yaml
	...
	spec:
	  containers:
	    ...
	    volumeMounts:
	    - name: secret-volume
	      mountPath: /mnt/secret-volume
	      readOnly: true
	  volumes:
	    - name: secret-volume
	      secret:
	       secretName: monsecret
	       defaultMode: 256
```

6/ Spécifier un chemin particulier pour un item (/mnt/secret-volume/my-group/my-username à la place de /mnt/secret-volume/username) et spécifier des autorisations différentes pour différents fichiers (ici, la valeur d'autorisation de 0777): 

```yaml
	volumes:
	  - name: foo
	    secret:
	      secretName: mysecret
	      items:
	      - key: username
		path: my-group/my-username
		mode: 511
```

- Si spec.volumes[].secret.items est utilisé, seules les clés spécifiées dans les items sont projetées. 
- Pour consommer toutes les clés du secret, elles doivent toutes être répertoriées dans le champ des items.
- Toutes les clés listées doivent exister dans le secret correspondant. Sinon, le volume n'est pas créé.


7/ Créer un pod qui a accès aux secret via des variables d'environnement:
Afin d'utiliser la variable secrète comme variable d'environnement, nous utiliserons env dans la section spec du fichier pod yaml.
```yaml
spec:
  containers:
   ...
   env:
   - name: ENVSECRET1
     valueFrom:
      secretKeyRef:
       name: usersecret
       key: username
   - name: ENVSECRET2
     valueFrom:
       secretKeyRef:
       name: passsecret
       key: password
```

Voir aussi: activer et configurer le cryptage des données:
https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/



### Utiliser les secerts avec les images
---------------------------------------------------------------------------------------------------------------
Pull une image d'un registre privé:
créer un pod qui utilise un secret pour extraire une image d'un registre Docker privé ou d'un référentiel.

vous devez vous authentifier auprès d'un registre afin d'extraire une image privée:

```bash
$  docker login 
```

Le processus de connexion crée ou met à jour un fichier config.json contenant un jeton d'autorisation.

Créer un secret dans le cluster qui contient votre jeton d'autorisation
Créez ce secret, en le nommant regcred :
```bash
$ kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```
Vous avez correctement défini vos informations d'identification Docker dans le cluster sous la forme d'un secret appelé regcred .

consulter le format Secret au format YAML:
```bash
$ kubectl get secret regcred --output=yaml 
```
La valeur du champ .dockerconfigjson est une représentation base64 de vos informations d'identification Docker.

Pour comprendre ce qui se trouve dans le champ .dockerconfigjson , convertissez les données secrètes dans un format lisible:

```bash
$ kubectl get secret regcred --output="jsonpath={.data.\.dockerconfigjson}" | base64 -d
```
Notez que les données secrètes contiennent le jeton d'autorisation similaire à votre fichier local ```~/.docker/config.json```.


Créer un pod qui utilise votre secret:
Voici un fichier de configuration pour un pod qui doit avoir accès à vos informations d'identification Docker dans regcred :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred
```  
remplacez <your-private-image> par le chemin d'accès à une image dans un registre privé tel que:  janedoe/jdoe-private:v1 
Le champ imagePullSecrets du fichier de configuration spécifie que Kubernetes doit obtenir les informations d'identification d'un secret nommé regcred .

Créez un pod qui utilise votre secret et vérifiez que le pod est en cours d'exécution:
$ kubectl create -f my-private-reg-pod.yaml
$ kubectl get pod private-reg





---------------------------------------------------------------------------------------------------------------
## Security Context
---------------------------------------------------------------------------------------------------------------
1/ Créer un Pod et configurer un contexte de sécurité (mode privilége) pour l'ensemble de ses pods:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simplepod6
  namespace: tst
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
    #seLinuxOptions:
      #level: "s0:c123,c456"
      #supplementalGroups: [5678]
  containers:
  - name: container6
    image: centos
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 10;done"]
```
*runAsUser: spécifie que pour tous les conteneur du Pod, le premier processus s'exécute avec l'ID utilisateur 1000. 
*fsGroup: spécifie que l'ID de groupe 2000 est associé à tous les Conteneurs du Pod (L'ID de groupe est également associé au volume monté).


2/ Obtenez un shell et dressez la liste des processus en cours pour vérifier que les processus s'exécutent en tant qu'utilisateur 1000
```bash
$  kubectl exec -it simplepod6 -- sh 
$  ps aux 
```

3/ Créer un fichier dans le répertoire de votre volume et vérifier la valeur de l'ID de groupe

4/ affichez les capacités du processus 1:
```bash
$ cd /proc/1
```


5/ Faites de même pour un contenair:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simplepod6
  namespace: tst
spec:
  containers:
  - name: container6
    image: centos
    securityContext:
      privileged: true
      runAsUser: 2000
      seLinuxOptions:
        level: "s0:c123,c456"
      allowPrivilegeEscalation: false
      capabilities:
	add: ["NET_ADMIN", "SYS_TIME"]
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 10;done"]
```



---------------------------------------------------------------------------------------------------------------
##ReplicaSet
---------------------------------------------------------------------------------------------------------------
Utiliser un ReplicaSet pour déployer un conteneur répliqué. . 
*L'une des fonctionnalités clés des contrôleurs de réplication comprenait la fonctionnalité de «mise à jour progressive». Cette fonctionnalité permet de mettre à jour les pods gérés par les contrôleurs de réplication avec une interruption minimale / nulle du service fourni par ces pods. Pour ce faire, les instances des anciens pods sont mises à jour une par une. Cependant, les contrôleurs de réplication ont été critiqués pour leur impératif et leur manque de flexibilité. En tant que solution, les jeux de réplicas et les déploiements ont été introduits en remplacement des contrôleurs de réplication.

1/ Créer un ReplicaSet:
```yaml
apiVersion: apps/v1beta2
kind: ReplicaSet
metadata:
  name: myrps
spec:
  replicas: 3
  selector:  #choisi les pods géré par notre ReplicaSet "Pod dont le label correspondant à ce selector". Donc un Controller peut gérer des pods qu'il n'a pas explicitement créées.
    matchLabels:
      app: nginx-replicaset
  template: # la configuration spécifique pour les pods dans ReplicaSet
    metadata:
      labels:
        app: nginx-replicaset
    spec:
      containers:  #règles d’exécution des conteneurs Docker présent de les pods
      - name: my-container 
        image: nginx  
        imagePullPolicy: IfNotPresent     # par défaut "IfNotPresent", oblige à extraire une image si elle n'existe pas
	#command: ['sh', '-c', 'echo Hello Kubernetes! &amp;&amp; sleep 3600']
        ports: #: spécifie le port utilisé par le conteneur
        - containerPort: 80
 ```  
 
*un modèle de pod dans un ReplicaSet doit spécifier les labels appropriées et une stratégie de redémarrage (la seule valeur autorisée est Always).
* Un ReplicaSet gère tous les pods avec des labels correspondant au sélecteur. Il ne fait pas la distinction entre les pods créés ou supprimés et les pods créés ou supprimés par une autre personne ou un autre processus. Cela permet de remplacer le ReplicaSet sans affecter les pods en cours d'exécution.
*vous ne devez pas créer de pod dont les labels correspondent à ce sélecteur, ni directement, ni avec un autre ReplicaSet, ni avec un autre contrôleur, tel qu'un déploiement. Si vous le faites, le ReplicaSet pense avoir créé les autres pods. Kubernetes ne vous empêche pas de le faire.
* Pour les Labels, veillez à ne pas les chevaucher avec d'autres contrôleurs.


2/ Afficher les détails du ReplicaSet:
```bash
$ kubectl describe rs/MyReplicaSet
$ kubectl describe rs frontend
```   

3/ Supprimer le ReplicaSet avec tous ces Pods:
```bash
$ kubectl delete rs frontend
```   

4/ Supprimer juste un ReplicaSet sans affecter aucun de ses pods:
```bash
$ kubectl delete rs frontend --cascade=false
```

5/ Isoler les pods d'un ReplicaSet:
Les pods peuvent être supprimés d'un ReplicaSet en modifiant leurs étiquettes.Les pods ainsi supprimés seront automatiquement remplacés (en supposant que le nombre de réplicas ne soit pas également modifié).


6/ Scaling d'un ReplicaSet:
Mettre à jour le champ .spec.replicas. Le contrôleur ReplicaSet s'assure qu'un nombre souhaité de pods avec un sélecteur correspondant au Labels sont disponibles et opérationnels.


7/ Créer le HPA défini qui met automatiquement à l'échelle le ReplicaSet cible en fonction de l'utilisation du CPU par les pods répliqués. Un ReplicaSet peut être Auto-Scaled par un HPA. 

```yaml
apiVersion:   autoscaling/v1 
  kind:   HorizontalPodAutoscaler 
  metadata: 
    name:   frontend-scaler 
  spec: 
    scaleTargetRef: 
      kind:   ReplicaSet 
      name:   frontend 
    minReplicas:   3 
    maxReplicas:   10 
    targetCPUUtilizationPercentage:   50
```

Ou, utiliser la commande kubectl autoscale pour accomplir la même chose que le HPA (Déploiement, Replica Set, Replication Controller)
```bash
$ kubectl autoscale (-f FILENAME | TYPE NAME | TYPE/NAME) [--min = MINPODS] --max = MAXPODS [--cpu-percent = CPU] [flags]
$ kubectl autoscale rs frontend --max=10
$ kubectl autoscale deployment foo --min=2 --max=10
```



---------------------------------------------------------------------------------------------------------------
## SERVICES
---------------------------------------------------------------------------------------------------------------
Lorsque k8s démarre un conteneur, il fournit des variables d'environnement pointant vers tous les services en cours d'exécution 
Si un service existe, tous les conteneurs recevront les variables
Ne spécifiez pas de hostPort pour un Pod, sauf si cela est absolument nécessaire. (Limite le nombre d'endroits où le Pod peut être planifié, car chaque hostIP < hostIP , hostPort , protocol > doit être unique)
 Si vous ne spécifiez pas explicitement le hostIP et le protocol , k8s utilise 0.0.0.0 comme hostIP par défaut et TCP comme protocol
Si vous avez seulement besoin d'accéder au port à des fins de débogage, vous pouvez utiliser le proxy kubectl port-forward
Si vous avez explicitement besoin d'exposer le port d'un Pod sur le nœud, envisagez d'utiliser un service NodePort avant de recourir à hostPort .
Évitez d'utiliser hostNetwork , pour les mêmes raisons que hostPort .
Utilisez les services sans ClusterIP  pour faciliter la découverte du service lorsque vous n'avez pas besoin de l'équilibrage.


1/ Service avec ou sans Selector:
```yaml
apiVersion: v1
kind: Service
metadata:
   name: My_Service
spec:
   selector: # falcultatif: Contraint à créer un Endpoint pour transférer le trafic
      application: "My Application"  
   ports:
   - port: 8080
   targetPort: 31999
```
*Dans cet exemple, nous avons un sélecteur; Pour transférer le trafic, nous devons donc créer manuellement un EndPoint
-créer un EndPoint qui acheminera le trafic vers le node final défini comme "192.168.168.40:8080".
```yaml
apiVersion: v1
kind: Endpoints
metadata:
   name: Tutorial_point_service
subnets:
   address:
      "ip": "192.168.168.40" -------------------> (Selector)
   ports:
      - port: 8080
```

2/ Service multi-ports:
```yaml
piVersion: v1
kind: Service
metadata:
   name: Tutorial_point_service
spec:
   selector:
      application: “My Application”
   ClusterIP: 10.3.0.12
   ports:
      -name: http
      protocol: TCP
      port: 80
      targetPort: 31999
   -name:https
      Protocol: TCP
      Port: 443
      targetPort: 31998
```      
*CLUSTERIP: Expose (restreindre) le service a l'interieur du cluster.       
      
      
3/ Créer un service complet "NodePort". 
*Un service ClusterIP, auquel ce service "NodePort" acheminera les flux est automatiquement créé. Le service est accéssible de l'extérieur à l'aide de :  NodeIP:NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
   name: My-service
   labels:
      k8s-app: appname
spec:
   type: NodePort   #Expose le service sur un port statique du node
   ports:
   - port: 8080
      nodePort: 31999
      name: Name-NodePord-Service
      #clusterIP: 10.10.10.10
   selector:
      k8s-app: appname
      component: nginx
      env: env_name
```   



---------------------------------------------------------------------------------------------------------------
##Controller Deployment
---------------------------------------------------------------------------------------------------------------
Bien que les ensembles des Controller aient toujours la capacité de gérer les pods et d’échelonner les instances de certains pods, ils ne peuvent pas effectuer de mise à jour propagée ni d’autres fonctionnalités. La méthode pour créer une application répliquée consiste à utiliser un déploiement, qui à son tour utilise un ReplicaSet. Le Deployment est un objet API de niveau supérieur qui met à jour ses ReplicaSets sous-jacents et leurs Pods de la même manière que kubectl rolling-update .

Ils ont la capacité de mettre à jour le jeu de réplicas et sont également capables de revenir à la version précédente. Ils fournissent de nombreuses fonctionnalités mises à jour de matchLabels et de sélecteurs.

1/ Exécuter une application à l'aide d'un objet Kubernetes Deployment.
```yaml
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: Mydeployment
spec:
  selector:  # Utilisé pour déterminer les Pods du Cluster géré par ce controller Deployment
    matchLabels:
      app: nginx
  replicas: 2 
  template: 
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```


2/ Créez un déploiement basé sur le fichier YAML:
```bash
$ kubectl apply -f https://k8s.io/docs/tasks/run-application/deployment.yaml
ou
$ kubectl run Name-Pod –image=Image-registry:tag 
```


3/ Afficher des informations sur le déploiement:
```bash
$ kubctl get deployments
$ kubectl describe deployment nginx-deployment 
```


4/ Vérifier l'état du déploiement
```bash
$ kubectl rollout status deployment/nginx-deployment
```
Revenir au déploiement précédent
```bash
$ kubectl rollout undo deployment/Deployment –to-revision=2
```


5/ Mittre à jour la version d'image "1.8" utilisé par les Pods du Déployment et appliquer le nouveau fichier YAML. 
```bash
$ kubectl apply -f deployment-update.yaml 
ou 
$ kubectl set image deployment/Deployment tomcat=tomcat:6.0
```
Alternativement, nous pouvons edit le Déploiement et changer:
```bash
$ kubectl edit deployment/nginx
```


6/ Augmenter le "Scaling" en augmentant le nombre de réplicas (Pods) et appliquer les fichier Yaml:
```bash
kubectl apply -f deployment-scale.yaml
$ kubectl get pods -l app=nginx
ou
$ kubectl scale deployment nginx-deployment --replicas=10 
```

7/ Supprimer un déploiement:
```bash
$  kubectl delete deployment nginx-deployment 
```



A Voir: 
-comment exécuter une application avec état à instance unique à l'aide de PersistentVolume et d'un déploiement.:
https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/

-comment exécuter une application avec état répliquée à l'aide d'un contrôleur StatefulSet. L'exemple est une topologie mono-maître MySQL avec plusieurs esclaves exécutant une réplication asynchrone. Notez qu'il ne s'agit pas d'une configuration de production. 
https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/

-Utiliser un correctif de fusion stratégique pour mettre à jour un déploiement:
https://kubernetes.io/docs/tasks/run-application/update-api-object-kubectl-patch/



---------------------------------------------------------------------------------------------------------------
## NETWORK POLICY:
---------------------------------------------------------------------------------------------------------------

$ kubectl create deployment nginx --image=nginx

*Un podSelector vide sélectionne tous les pods de l'espace de noms.
* chaque politique NetworkPolicy comprend une liste policyTypes pouvant inclure Ingress , Egress ou les deux.Si aucun type de policyTypes n'est spécifié sur un NetworkPolicy, par défaut, Ingress sera toujours défini et Egress sera défini si NetworkPolicy a des règles de sortie.

```yaml
apiVersion: extensions/v1beta1
kind: NetworkPolicy
metadata:
   name: policyfrontend
   namespace: tst
spec:
   podSelector:
      matchLabels:
         role: backend
  policyTypes: 
  -   Ingress 
  -   Egress 
  ingress:	 
   - from:
      - podSelector:
         matchLabels:
            role: db
   ports:
      - protocol: TCP
         port: 6379
egress: 
  -   to: 
    -   ipBlock: 
        cidr:   10.0.0.0/24 
    ports: 
    -   protocol:   TCP 
      port:   5978
```

- La NetworkPolicy du nom de "policyfrontend", isole les pods identifiés par le label "role=frontend" dans le namespace "tst" pour le trafic entrant "Ingress" et sortant "Egress". Sur l'ensemble des pods selectionné, elle identifie ceux contenant le label "role=db" et leurs autorise les connexions entrante sur le port TCP 6379.

voir: https://kubernetes.io/docs/concepts/services-networking/network-policies/



---------------------------------------------------------------------------------------------------------------
##JOB
---------------------------------------------------------------------------------------------------------------

1/ Creer un Job
```yaml
apiVersion: v1
kind: Job   (création d’un Job de type Pod)
metadata:
   name: py
   spec:
   template:
      metadata
      name: py -------> 2
      spec:
         containers:
            - name: py ------------------------> 3
            image: python----------> 4
            command: ["python", "SUCCESS"]
            restartPocliy: Never --------> 5
```
∙ kind: Job → Nous avons défini le type de Job qui indiquera à kubectlt que le fichier yaml utilisé doit créer un module de type de travail.
∙ Nom: py → C'est le nom du modèle que nous utilisons et la spécification définit le modèle.
∙ nom: py → nous avons donné le nom py dans les spécifications de conteneur, ce qui permet d'identifier le pod qui sera créé.
Image: python → l'image que nous allons extraire pour créer le conteneur qui s'exécutera à l'intérieur du pod.
∙ restartPolicy: Jamais → Cette condition de redémarrage de l'image est donnée comme jamais, ce qui signifie que si le conteneur est tué ou s'il est faux, il ne redémarrera pas tout seul.


2/ SCréer un scheduled Job
```yaml
apiVersion: v1
kind: Job
metadata:
   name: py
spec:
   schedule: h/30 * * * * ? -------------------> 1
   template:
      metadata
         name: py
      spec:
         containers:
         - name: py
         image: python
         args:
/bin/sh -------> 2
-c
ps –eaf ------------> 3
restartPocliy: OnFailure
```

 - hschedule: Pour planifier l'exécution du Job toutes les 30 minutes.
 - /bin/sh: entrer dans le conteneur avec /bin/sh
 - ps –eaf: Exécute la commande ps -eaf sur la machine et répertorie tous les processus en cours d'exécution dans un conteneur.
 
 
Pour lister tous les pods appartenant à un job sous une forme lisible par une machine:
```bash
$ pods=$(kubectl get pods --selector=job-name=pi --output=jsonpath={.items..metadata.name})
$ echo $pods
```

L'option --output=jsonpath spécifie une expression qui récupère juste le nom de chaque pod dans la liste renvoyée.
```bash
$ kubectl logs $pods
```




4/ Créer un CronJob qui exécute chaque minute un travail simple pour imprimer l'heure actuelle et ensuite dire bonjour. Toutes les modifications apportées à un travail cron, en particulier son .spec , ne seront appliquées qu'à la prochaine exécution.

Créer un CronJob:
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```
Vous pouvez également utiliser kubectl run pour créer un travail cron sans écrire la configuration complète:
```bash
$ kubectl run hello --schedule="*/1 * * * *" --restart=OnFailure --image=busybox -- /bin/sh -c "date; echo Hello from the Kubernetes cluster"
```

Vous devriez voir que "bonjour" a programmé avec succès un travail à l'heure spécifiée dans LAST-SCHEDULE .
Afficher les CronJob:
```bash
$ kubectl get cronjob 
$ kubectl get jobs –watch
```

Supprimer le CronJob:
```bash
$ kubectl delete cronjob hello
```




---------------------------------------------------------------------------------------------------------------
## Init Container
---------------------------------------------------------------------------------------------------------------

Configurer l'initialisation du pod:
comment utiliser un Init Container pour initialiser un Pod avant l'exécution d'un conteneur d'application.
créez un pod avec un conteneur d'applications et un conteneur d'initialisation. 
Le conteneur init est exécuté avant le démarrage du conteneur d'application.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  # These containers are run during pod initialization
  initContainers:
  - name: install
    image: busybox
    command:
    - wget
    - "-O"
    - "/work-dir/index.html"
    - http://kubernetes.io
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  dnsPolicy: Default
  volumes:
  - name: workdir
    emptyDir: {}
```

vous pouvez voir que le pod a un volume que le conteneur init et le conteneur d'application partagent.
Le conteneur init monte le volume partagé dans le ```/usr/share/nginx/html /work-dir``` , et le conteneur d'application monte le volume partagé dans ```/usr/share/nginx/html```. 
Le conteneur init exécute la commande suivante, puis se termine:  wget -O /work-dir/index.html http://kubernetes.io 
Notez que le conteneur init écrit le fichier index.html dans le répertoire racine du serveur nginx.



2/ Pod simple qui a deux Init Containers. Le premier attend myservice et le second attend mydb . Une fois les deux conteneurs terminés, le pod commencera.

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']




---
## PodPreset

Injecter des informations dans des pods à l'aide d'un PodPreset:
Vous pouvez utiliser un objet podpreset pour injecter des informations telles que des secrets, des montages de volume et des variables d'environnement, etc. dans des pods au moment de la création.

Créer un préréglage de pod:
```yaml
apiVersion: settings.k8s.io/v1alpha1
kind: PodPreset
metadata:
  name: allow-database
spec:
  selector:
    matchLabels:
      role: frontend
  env:
    - name: DB_PORT
      value: "6379"
  volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
    - name: cache-volume
      emptyDir: {}
```

Ceci est un exemple pour montrer comment une spécification de Pod est modifiée par le préréglage de Pod qui définit une variable ConfigMap pour les variables d'environnement:

Spécification de pod soumise par l'utilisateur:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: website
  labels:
    app: website
    role: frontend
spec:
  containers:
    - name: website
      image: nginx
      ports:
        - containerPort: 80
```


Utilisateur soumis ConfigMap :

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: etcd-env-config
data:
  number_of_members: "1"
  initial_cluster_state: new
  initial_cluster_token: DUMMY_ETCD_INITIAL_CLUSTER_TOKEN
  discovery_token: DUMMY_ETCD_DISCOVERY_TOKEN
  discovery_url: http://etcd_discovery:2379
  etcdctl_peers: http://etcd:2379
  duplicate_key: FROM_CONFIG_MAP
  REPLACE_ME: "a value"
```

Exemple de préréglage de pod:

```yaml
apiVersion: settings.k8s.io/v1alpha1
kind: PodPreset
metadata:
  name: allow-database
spec:
  selector:
    matchLabels:
      role: frontend
  env:
    - name: DB_PORT
      value: "6379"
    - name: duplicate_key
      value: FROM_ENV
    - name: expansion
      value: $(REPLACE_ME)
  envFrom:
    - configMapRef:
        name: etcd-env-config
  volumeMounts:
    - mountPath: /cache
      name: cache-volume
    - mountPath: /etc/app/config.json
      readOnly: true
      name: secret-volume
  volumes:
    - name: cache-volume
      emptyDir: {}
    - name: secret-volume
      secret:
         secretName: config-details
```

Pod spec après admission controller:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: website
  labels:
    app: website
    role: frontend
  annotations:
    podpreset.admission.kubernetes.io/podpreset-allow-database: "resource version"
spec:
  containers:
    - name: website
      image: nginx
      volumeMounts:
        - mountPath: /cache
          name: cache-volume
        - mountPath: /etc/app/config.json
          readOnly: true
          name: secret-volume
      ports:
        - containerPort: 80
      env:
        - name: DB_PORT
          value: "6379"
        - name: duplicate_key
          value: FROM_ENV
        - name: expansion
          value: $(REPLACE_ME)
      envFrom:
        - configMapRef:
            name: etcd-env-config
  volumes:
    - name: cache-volume
      emptyDir: {}
    - name: secret-volume
      secret:
         secretName: config-details
```


Suppression d'un préréglage de pod:
```
$ kubectl delete podpreset allow-database podpreset "allow-database" deleted 
```




---------------------------------------------------------------------------------------------------------------
## ConfigMap
---------------------------------------------------------------------------------------------------------------
Configurer un pod pour utiliser un fichier ConfigMap:
ConfigMaps vous permet de découpler les artefacts de configuration du contenu de l'image pour garder les applications conteneurisées portables.

Exemple: La configuration dynamique de Kubelet vous permet de modifier la configuration de chaque Kubelet dans un cluster Kubernetes actif en déployant un ConfigMap et en configurant chaque nœud pour son utilisation.
https://kubernetes.io/docs/tasks/administer-cluster/reconfigure-kubelet/


comment créer des fichiers ConfigMaps et configurer des modules en utilisant des données stockées dans ConfigMaps.
créer des configmaps à partir de répertoires , de fichiers ou de valeurs littérales :
```bash
$ kubectl create configmap <map-name> <data-source>
```
- map-name est le nom que vous souhaitez attribuer à ConfigMap
- data-source est le répertoire, le fichier ou la valeur littérale dans laquelle les données doivent être dessinées.

La source de données correspond à une paire clé-valeur dans ConfigMap, où
key = le nom du fichier ou la clé que vous avez fournie sur la ligne de commande, et
value = le contenu du fichier ou la valeur littérale que vous avez fournie sur la ligne de commande.
Vous pouvez utiliser kubectl describe ou kubectl get pour kubectl get informations sur un ConfigMap.


Créer des ConfigMaps à partir de répertoires ou à partir de plusieurs fichiers du même répertoire.
```
$ kubectl create configmap game-config --from-file=https://k8s.io/docs/tasks/configure-pod-container/configmap/kubectl
$ kubectl describe configmaps game-config
$ kubectl get configmaps game-config -o yaml
```

```yaml
apiVersion: v1
data:
  game.properties: |
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
  ui.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
    how.nice.to.look=fairlyNice
kind: ConfigMap
metadata:
  creationTimestamp: 2016-02-18T18:52:05Z
  name: game-config
  namespace: default
  resourceVersion: "516"
  selfLink: /api/v1/namespaces/default/configmaps/game-config
  uid: b4952dc3-d670-11e5-8cd0-68f728db1985
```



Créer des ConfigMaps à partir de fichiers:
créer un fichier ConfigMap à partir d'un fichier individuel ou de plusieurs fichiers.
```
$ kubectl create configmap game-config-2 --from-file=https://k8s.io/docs/tasks/configure-pod-container/configmap/kubectl/game.properties
```
Vous pouvez passer l'argument --from-file plusieurs fois pour créer un fichier ConfigMap à partir de plusieurs sources de données.

Utilisez l'option --from-env-file pour créer un --from-env-file ConfigMap à partir d'un fichier env, par exemple:
```
$ kubectl create configmap game-config-env-file --from-env-file=docs/tasks/configure-pod-container/game-env-file.properties
```

A VOIR !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/





---------------------------------------------------------------------------------------------------------------
## Handlers
---------------------------------------------------------------------------------------------------------------
https://kubernetes.io/docs/tasks/administer-cluster/out-of-resource/
Attacher des gestionnaires aux événements du cycle de vie des conteneurs: (Handlers)
Kubernetes prend en charge les événements postStart et preStop. 
Kubernetes envoie l'événement postStart immédiatement après le démarrage d'un conteneur et envoie l'événement preStop immédiatement avant la fin du conteneur.
créez un pod avec un conteneur. Le conteneur a des gestionnaires pour les événements postStart et preStop:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo
spec:
  containers:
  - name: lifecycle-demo-container
    image: nginx
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
      preStop:
        exec:
          command: ["/usr/sbin/nginx","-s","quit"]
```

vous pouvez voir que la commande postStart écrit un fichier de message dans le ```/usr/share``` du conteneur. 
La commande preStop arrête nginx avec élégance. Ceci est utile si le conteneur est arrêté à cause d'un échec.
Le gestionnaire postStart s'exécute de manière asynchrone par rapport au code du conteneur, mais la gestion par Kubernetes du conteneur se bloque jusqu'à la fin du gestionnaire postStart. 
Le statut du conteneur n'est pas défini sur RUNNING tant que le gestionnaire post-démarrage n'est pas terminé.
Kubernetes envoie uniquement l'événement preStop lorsqu'un module est terminé . Cela signifie que le hook preStop n'est pas appelé lorsque le pod est terminé .




