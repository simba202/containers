---

copyright:
  years: 2014, 2018
lastupdated: "2018-09-11"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:download: .download}



# Tutoriel : Utilisation des règles réseau Calico pour bloquer le trafic
{: #policy_tutorial}

Par défaut, les services Kubernetes NodePort, LoadBalancer et Ingress rendent votre application accessible au public et à toutes les interfaces réseau du cluster publiques et privées. La règle réseau Calico par défaut `allow-node-port-dnat` autorise le trafic entrant en provenance des services NodePort, LoadBalancer et Ingress vers les pods d'application exposés par ces services. Kubernetes utilise la conversion d'adresse réseau de destination (DNAT) pour transférer les demandes de service aux pods appropriés.

Cependant, pour des raisons de sécurité, vous pouvez être amené à autoriser le trafic vers les services de réseau uniquement à partir de certaines adresses IP source. Vous pouvez utiliser des [règles Calico Pre-DNAT ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](https://docs.projectcalico.org/v3.1/getting-started/bare-metal/policy/pre-dnat) pour inscrire sur liste blanche ou liste noire le trafic en provenance ou à destination de certaines adresses IP. Les règles Pre-DNAT empêchent le trafic d'atteindre vos applications car elles sont appliquées avant que Kubernetes utilise la fonction DNAT standard pour acheminer le trafic vers les pods. Lorsque vous créez des règles Calico Pre-DNAT, vous déterminez les adresses IP source à inscrire sur liste blanche ou sur liste noire. Dans la plupart des cas, l'inscription sur liste blanche fournit la configuration la plus sécurisée car tout le trafic est bloqué à l'exception du trafic en provenance d'adresses IP source autorisées. En principe, l'utilisation d'une liste noire n'est utile que dans les scénarios utilisés pour empêcher une attaque en provenance d'un petit groupe d'adresses IP.

Dans ce scénario, vous jouez le rôle d'un administrateur réseau pour une entreprise de relations publiques et vous constatez l'arrivée de trafic inhabituel dans vos applications. Les leçons de ce tutoriel vous guident dans le processus de création d'un modèle d'application de serveur Web, exposant l'application à l'aide d'un service LoadBalancer et protégeant l'application contre le trafic inhabituel indésirable avec des règles Calico de liste blanche et de liste noire.

## Objectifs

- Apprendre à bloquer tout le trafic entrant sur tous les services NodePorts en créant une règle Pre-DNAT de poids fort.
- Apprendre à autoriser des adresses IP source sur liste blanche à accéder à l'adresse IP publique et au port du service LoadBalancer en créant une règle Pre-DNAT de poids faible. Les règles de poids faible remplacent les règles de poids fort.
- Apprendre à bloquer les adresses IP source sur liste noire pour les empêcher d'accéder à l'adresse IP publique et au port en créant une règle Pre-DNAT de poids faible.

## Durée
1 heure

## Public
Ce tutoriel est destiné aux développeurs de logiciel et aux administrateurs réseau souhaitant gérer le trafic réseau vers une application.

## Conditions prérequises

- [Créez un cluster en version 1.10](cs_clusters.html#clusters_ui) ou [mettez à jour un cluster à la version 1.10](cs_versions.html#cs_v110). Vous devez disposer d'un cluster Kubernetes de version 1.10 ou ultérieure pour utiliser l'interface de ligne de commande de Calico 3.1.1 et la syntaxe des règles Calico v3 dans ce tutoriel.
- [Ciblez votre interface CLI sur le cluster](cs_cli_install.html#cs_cli_configure).
- [Installez et configurez l'interface de ligne de commande de Calico](cs_network_policy.html#1.10_install).
- [Assurez-vous de disposer du rôle de plateforme **Editeur**, **Opérateur** ou **Administrateur**](cs_users.html#add_users_cli).

<br />


## Leçon 1 : Déploiement d'une application et exposition de cette application à l'aide d'un service LoadBalancer
{: #lesson1}

La première leçon vous montre comment est exposée votre application depuis plusieurs adresses IP et ports et par où passe le trafic public pour atteindre votre cluster.
{: shortdesc}

Commencez par déployer un modèle d'application de serveur Web à utiliser tout au long de ce tutoriel. Le serveur Web `echoserver` présente les données de la connexion établie avec le cluster à partir du client et vous laisse tester l'accès au cluster de l'entreprise de relations publiques (RP). Exposez ensuite l'application en créant un service LoadBalancer. Un service LoadBalancer rend votre application accessible via l'adresse IP du service LoadBalancer et les services NodePorts des noeuds worker.

L'image suivante montre comment l'application de serveur Web sera exposée sur Internet par les services NodePort et LoadBalancer publics à la fin de la leçon 1 :

<img src="images/cs_tutorial_policies_Lesson1.png" width="450" alt="A la fin de la leçon 1, l'application de serveur Web est exposée sur Internet par les services NodePort et LoadBalancer publics." style="width:450px; border-style: none"/>

1. Déployez le modèle d'application de serveur Web. Lorsqu'une connexion à l'application de serveur Web est établie, l'application répond avec les en-têtes HTTP qu'elle a reçus dans la connexion.
    ```
    kubectl run webserver --image=k8s.gcr.io/echoserver:1.10 --replicas=3
    ```
    {: pre}

2. Vérifiez que la zone **STATUS** des pods de l'application de serveur Web a la valeur `Running`.
    ```
    kubectl get pods -o wide
    ```
    {: pre}

    Exemple de sortie :
    ```
    NAME                         READY     STATUS    RESTARTS   AGE       IP               NODE
    webserver-855556f688-6dbsn   1/1       Running   0          1m        172.30.xxx.xxx   10.176.48.78
    webserver-855556f688-76rkp   1/1       Running   0          1m        172.30.xxx.xxx   10.176.48.78
    webserver-855556f688-xd849   1/1       Running   0          1m        172.30.xxx.xxx   10.176.48.78
    ```
    {: screen}

3. Pour exposer l'application sur l'Internet public, créez un fichier de configuration de service LoadBalancer nommé `webserver.yaml` dans un éditeur de texte.
    ```
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        run: webserver
      name: webserver-lb
      namespace: pr-firm
    spec:
      externalTrafficPolicy: Cluster
      ports:
      - name: webserver-port
        port: 80
        protocol: TCP
        targetPort: 8080
      selector:
        run: webserver
      type: LoadBalancer
    ```
    {: codeblock}

4. Déployez le service LoadBalancer.
    ```
    kubectl apply -f filepath/webserver-lb.yaml
    ```
    {: pre}

5. Vérifiez que vous disposez d'un accès public à l'application exposée par le service LoadBalancer sur votre ordinateur.

    1. Obtenez l'adresse IP publique (**EXTERNAL-IP**) du service LoadBalancer.
        ```
        kubectl get svc -o wide
        ```
        {: pre}

        Exemple de sortie :
        ```
        NAME           CLUSTER-IP       EXTERNAL-IP        PORT(S)        AGE       SELECTOR
        webserver-lb   172.21.xxx.xxx   169.xx.xxx.xxx     80:31024/TCP   2m        run=webserver
        ```
        {: screen}

    2. Créez un fichier aide-mémoire et copiez l'adresse IP du service LoadBalancer dans ce fichier texte. Cet aide-mémoire vous aidera à retrouver plus rapidement les valeurs requises dans les leçons suivantes.

    3. Vérifiez que vous disposez d'un accès public à l'adresse IP externe du service LoadBalancer.
        ```
        curl --connect-timeout 10 <loadbalancer_IP>:80
        ```
        {: pre}

        L'exemple de sortie suivante confirme que le service LoadBalancer expose votre application sur l'adresse IP publique `169.1.1.1` du service LoadBalancer. Le pod d'application `webserver-855556f688-76rkp` reçoit la demande curl :
        ```
        Hostname: webserver-855556f688-76rkp
        Pod Information:
            -no pod information available-
        Server values:
            server_version=nginx: 1.13.3 - lua: 10008
        Request Information:
            client_address=1.1.1.1
            method=GET
            real path=/
            query=
            request_version=1.1
            request_scheme=http
            request_uri=http://169.1.1.1:8080/
        Request Headers:
            accept=*/*
            host=169.1.1.1
            user-agent=curl/7.54.0
        Request Body:
            -no body in request-
        ```
        {: screen}

6. Vérifiez que vous disposez d'un accès public à l'application exposée par le service NodePort sur votre ordinateur. Un service LoadBalancer rend votre application accessible via l'adresse IP du service LoadBalancer et les services NodePorts des noeuds worker.

    1. Obtenez la valeur NodePort affectée par le service LoadBalancer aux noeuds worker. La valeur de NodePort est comprise entre 30000 et 32767.
        ```
        kubectl get svc -o wide
        ```
        {: pre}

        Dans la sortie de l'exemple suivant, la valeur de NodePort est `31024` :
        ```
        NAME           CLUSTER-IP       EXTERNAL-IP        PORT(S)        AGE       SELECTOR
        webserver-lb   172.21.xxx.xxx   169.xx.xxx.xxx     80:31024/TCP   2m        run=webserver
        ```
        {: screen}  

    2. Obtenez l'**adresse IP publique** d'un noeud worker.
        ```
        ibmcloud ks workers <cluster_name>
        ```
        {: pre}

        Exemple de sortie :
        ```
        ID                                                 Public IP        Private IP     Machine Type        State    Status   Zone    Version   
        kube-dal10-cr18e61e63c6e94b658596ca93d087eed9-w1   169.xx.xxx.xxx   10.176.48.67   u2c.2x4.encrypted   normal   Ready    dal10   1.10.7_1513*   
        kube-dal10-cr18e61e63c6e94b658596ca93d087eed9-w2   169.xx.xxx.xxx   10.176.48.79   u2c.2x4.encrypted   normal   Ready    dal10   1.10.7_1513*   
        kube-dal10-cr18e61e63c6e94b658596ca93d087eed9-w3   169.xx.xxx.xxx   10.176.48.78   u2c.2x4.encrypted   normal   Ready    dal10   1.10.7_1513*   
        ```
        {: screen}

    3. Copiez l'adresse IP publique du noeud worker et la valeur de NodePort dans votre aide-mémoire pour les utiliser dans les leçons suivantes.

    4. Vérifiez que vous pouvez accéder à l'adresse IP publique du noeud worker via le service NodePort.
        ```
        curl  --connect-timeout 10 <worker_IP>:<NodePort>
        ```
        {: pre}

        L'exemple de sortie suivant confirme que la demande transmise à votre application provient de l'adresse IP privée `10.1.1.1` du noeud worker et du service NodePort `31024`. Le pod d'application `webserver-855556f688-xd849` a reçu la demande curl :
        ```
        Hostname: webserver-855556f688-xd849
        Pod Information:
            -no pod information available-
        Server values:
            server_version=nginx: 1.13.3 - lua: 10008
        Request Information:
            client_address=1.1.1.1
            method=GET
            real path=/
            query=
            request_version=1.1
            request_scheme=http
            request_uri=http://10.1.1.1:8080/
        Request Headers:
            accept=*/*
            host=10.1.1.1:31024
            user-agent=curl/7.60.0
        Request Body:
            -no body in request-
        ```
        {: screen}

A ce stade, votre application est exposée à partir de plusieurs ports et adresses IP. La plupart de ces adresses IP sont internes au cluster et sont accessibles uniquement via le réseau privé. Seuls le service NodePort public et le port LoadBalancer public sont exposés sur l'Internet public.

Ensuite, vous pouvez commencer à créer et appliquer des règles Calico pour bloquer le trafic public.

## Leçon 2 : Blocage de tout le trafic entrant sur tous les services NodePorts
{: #lesson2}

Pour sécuriser le cluster de l'entreprise de relations publiques, vous devez bloquer l'accès public aux services LoadBalancer et NodePorts exposant votre application. Commencez par bloquer l'accès aux services NodePorts. L'image suivante montre comment le trafic sera autorisé vers le service LoadBalancer mais pas vers les services NodePorts à la fin de la leçon 2 :

<img src="images/cs_tutorial_policies_Lesson2.png" width="450" alt="A la fin de la leçon 2, l'application de serveur Web est exposée sur Internet uniquement par le service LoadBalancer public." style="width:450px; border-style: none"/>

1. Dans un éditeur de texte, créez une règle Pre-DAT de poids fort nommée `deny-nodeports.yaml` pour refuser le trafic entrant TCP et UDP provenant d'une adresse IP source vers tous les services NodePorts.
    ```
    apiVersion: projectcalico.org/v3
    kind: GlobalNetworkPolicy
    metadata:
      name: deny-nodeports
    spec:
      applyOnForward: true
      ingress:
      - action: Deny
            destination:
              ports:
          - 30000:32767
            protocol: TCP
            source: {}
      - action: Deny
            destination:
              ports:
          - 30000:32767
        protocol: UDP
        source: {}
      preDNAT: true
      selector: ibm.role=='worker_public'
      order: 1100
      types:
      - Ingress
    ```
    {: codeblock}

2. Appliquez la règle.
    - Linux :

      ```
      calicoctl apply -f filepath/deny-nodeports.yaml
      ```
      {: pre}

    - Windows et OS X :

      ```
      calicoctl apply -f filepath/deny-nodeports.yaml --config=filepath/calicoctl.cfg
      ```
      {: pre}
  Exemple de sortie :
  ```
  Successfully applied 1 'GlobalNetworkPolicy' resource(s)
  ```
  {: screen}

3. En utilisant les valeurs de votre aide-mémoire, vérifiez que vous ne disposez d'aucun accès public à l'adresse IP publique et au service NodePort du noeud worker.
    ```
    curl  --connect-timeout 10 <worker_IP>:<NodePort>
    ```
    {: pre}

    La connexion arrive à expiration car la règle Calico que vous avez créée bloque le trafic vers les services NodePorts.
    ```
    curl: (28) Connection timed out after 10016 milliseconds
    ```
    {: screen}

4. Modifiez la règle externalTrafficPolicy du service LoadBalancer que vous avez créée dans la leçon précédente en remplaçant `Cluster` par `Local`. `Local` garantit que l'adresse IP source de votre système est conservée lorsque vous utilisez curl sur l'adresse IP du service LoadBalancer à l'étape suivante.
    ```
    kubectl patch svc webserver-lb -p '{"spec":{"externalTrafficPolicy":"Local"}}'
    ```
    {: pre}

5. En utilisant la valeur de votre aide-mémoire, vérifiez que vous bénéficiez toujours de l'accès public à l'adresse IP externe du service LoadBalancer.
    ```
    curl --connect-timeout 10 <loadbalancer_IP>:80
    ```
    {: pre}

    Exemple de sortie :
    ```
    Hostname: webserver-855556f688-76rkp
    Pod Information:
        -no pod information available-
    Server values:
        server_version=nginx: 1.13.3 - lua: 10008
    Request Information:
        client_address=1.1.1.1
        method=GET
        real path=/
        query=
        request_version=1.1
        request_scheme=http
        request_uri=http://<loadbalancer_IP>:8080/
    Request Headers:
        accept=*/*
        host=<loadbalancer_IP>
        user-agent=curl/7.54.0
    Request Body:
        -no body in request-
    ```
    {: screen}
    Dans la section `Request Information` de la sortie, notez que l'adresse IP source est, par exemple, `client_address=1.1.1.1`. L'adresse IP source est l'adresse IP publique du système que vous utilisez pour exécuter la commande curl. Autrement, si vous vous connectez à Internet via un proxy ou un réseau privé virtuel (VPN), le proxy ou le VPN peut rendre illisible l'adresse IP réelle de votre système. Dans les deux cas, le service LoadBalancer voit l'adresse IP source de votre système en tant qu'adresse IP client.

6. Copiez l'adresse IP source de votre système (`client_address=1.1.1.1` figurant dans la sortie de l'étape précédente) dans votre aide-mémoire pour l'utiliser dans les leçons suivantes.

Parfait ! A ce stade, votre application est exposée sur l'Internet public uniquement à partir du port LoadBalancer public. Le trafic vers les services NodePorts publics est bloqué. Vous avez verrouillé en partie votre cluster par rapport au trafic indésirable.

Ensuite, vous pouvez créer et appliquer des règles Calico pour inscrire sur liste blanche le trafic en provenance de certaines adresses IP source.

## Leçon 3 : Autorisation de trafic entrant à partir d'une adresse IP sur liste blanche vers le service LoadBalancer
{: #lesson3}

A présent, vous décidez de bloquer l'intégralité du trafic vers le cluster de l'entreprise de relations publiques et de tester l'accès en inscrivant uniquement l'adresse IP de votre ordinateur sur liste blanche.
{: shortdesc}

Tout d'abord, en plus des services NodePorts, vous devez bloquer tout le trafic entrant vers le service LoadBalancer exposant l'application. Ensuite, vous pouvez créer une règle pour inscrire l'adresse IP de votre système sur liste blanche. A la fin de la leçon 3, tout le trafic vers les services NodePorts et LoadBalancer publics est verrouillé et seul le trafic en provenance de l'adresse IP de votre système sur liste blanche est autorisé :
<img src="images/cs_tutorial_policies_L3.png" width="600" alt="L'application de serveur Web et exposée par le service LoadBalancer public uniquement sur l'adresse IP de votre système." style="width:600px; border-style: none"/>

1. Dans un éditeur de texte, créez une règle Pre-DNAT de poids fort nommé `deny-lb-port-80.yaml` pour refuser tout le trafic TCP et UDP entrant en provenance de n'importe quelle adresse IP source vers l'adresse IP et le port du service LoadBalancer. Remplacez `<loadbalancer_IP>` par l'adresse IP publique du service LoadBalancer consignée dans votre aide-mémoire.
    ```
    apiVersion: projectcalico.org/v3
    kind: GlobalNetworkPolicy
    metadata:
      name: deny-lb-port-80
    spec:
      applyOnForward: true
      ingress:
      - action: Deny
        destination:
          nets:
          - <loadbalancer_IP>/32
          ports:
          - 80
        protocol: TCP
        source: {}
      - action: Deny
        destination:
          nets:
          - <loadbalancer_IP>/32
          ports:
          - 80
        protocol: UDP
        source: {}
      preDNAT: true
      selector: ibm.role=='worker_public'
      order: 1100
      types:
      - Ingress
    ```
    {: codeblock}

2. Appliquez la règle.
    - Linux :

      ```
      calicoctl apply -f filepath/deny-lb-port-80.yaml
      ```
      {: pre}

    - Windows et OS X :

      ```
      calicoctl apply -f filepath/deny-lb-port-80.yaml --config=filepath/calicoctl.cfg
      ```
      {: pre}

3. En utilisant la valeur de votre aide-mémoire, vérifiez que vous ne pouvez plus accéder à l'adresse IP publique du service LoadBalancer. La connexion arrive à expiration car la règle Calico que vous avez créée bloque le trafic vers le service LoadBalancer.
    ```
    curl --connect-timeout 10 <loadbalancer_IP>:80
    ```
    {: pre}

4. Dans un éditeur de texte, créez une règle Pre-DNAT de poids faible nommée `whitelist.yaml` pour autoriser le trafic en provenance de l'adresse IP de votre système vers l'adresse IP et le port du service LoadBalancer. En utilisant les valeurs de votre aide-mémoire, remplacez `<loadbalancer_IP>` par l'adresse IP publique du service LoadBalancer et `<client_address>` par l'adresse IP publique de l'adresse IP source de votre système.
    ```
    apiVersion: projectcalico.org/v3
    kind: GlobalNetworkPolicy
    metadata:
      name: whitelist
    spec:
      applyOnForward: true
      ingress:
      - action: Allow
        destination:
          nets:
          - <loadbalancer_IP>/32
          ports:
          - 80
        protocol: TCP
        source:
          nets:
          - <client_address>/32
      preDNAT: true
      selector: ibm.role=='worker_public'
      order: 500
      types:
      - Ingress
    ```
    {: codeblock}

5. Appliquez la règle.
    - Linux :

      ```
      calicoctl apply -f filepath/whitelist.yaml
      ```
      {: pre}

    - Windows et OS X :

      ```
      calicoctl apply -f filepath/whitelist.yaml --config=filepath/calicoctl.cfg
      ```
      {: pre}
  L'adresse IP de votre système est désormais sur liste blanche.

6. En utilisant la valeur de votre aide-mémoire, vérifiez que vous pouvez désormais accéder à l'adresse IP publique du service LoadBalancer.
    ```
    curl --connect-timeout 10 <loadbalancer_IP>:80
    ```
    {: pre}

7. Si vous avez accès à un autre système ayant une adresse IP différente, essayez d'accéder au service LoadBalancer à partir de ce système.
    ```
    curl --connect-timeout 10 <loadbalancer_IP>:80
    ```
    {: pre}
    La connexion arrive à expiration car l'adresse IP de ce système ne figure pas sur la liste blanche.

A ce stade, tout le trafic vers les services NodePorts et LoadBalancer est bloqué. Seul le trafic en provenance de l'adresse IP de votre système sur liste blanche est autorisé.

## Leçon 4 : Refus d'accès entrant en provenance d'adresses IP en liste noire au service LoadBalancer
{: #lesson4}

Dans la leçon précédente, vous avez bloqué tout le trafic et inscrit uniquement quelques adresses IP sur liste blanche. Ce scénario fonctionne bien à des fins de test lorsque vous voulez limiter l'accès à quelques adresses IP source contrôlées. Cependant, l'entreprise de relations publiques disposent d'applications qui nécessitent une plus grande ouverture au public. Vous devez veiller à ce que tout le trafic soit autorisé sauf le trafic inhabituel que vous voyez en provenance de quelques adresses IP. L'utilisation d'une liste noire est utile dans un scénario de ce type car elle vous permet d'éviter toute attaque en provenance d'un petit groupe d'adresses IP.

Dans cette leçon, vous allez tester l'utilisation d'une liste noire pour bloquer le trafic en provenance de l'adresse IP source de votre propre système. A la fin de la leçon 4, tout le trafic vers les services NodePorts publics sera bloqué et tout le trafic vers le service LoadBalancer public sera autorisé. Seul le trafic en provenance de l'adresse IP de votre système sur liste noire vers le service LoadBalancer sera bloqué :
<img src="images/cs_tutorial_policies_L4.png" width="600" alt="L'application de serveur Web est exposée sur Internet par le service LoadBalancer public. Seul le trafic en provenance de l'adresse IP de votre système est bloqué." style="width:600px; border-style: none"/>

1. Supprimez les règles d'inscription sur liste blanche que vous avez créées dans la leçon précédente.
    - Linux :
      ```
      calicoctl delete GlobalNetworkPolicy deny-lb-port-80
      ```
      {: pre}
      ```
      calicoctl delete GlobalNetworkPolicy whitelist
      ```
      {: pre}

    - Windows et OS X :
      ```
      calicoctl delete GlobalNetworkPolicy deny-lb-port-80 --config=filepath/calicoctl.cfg
      ```
      {: pre}
      ```
      calicoctl delete GlobalNetworkPolicy whitelist --config=filepath/calicoctl.cfg
      ```
      {: pre}

    Désormais, tout le trafic TCP et UDP entrant en provenance de n'importe quelle source IP vers l'adresse IP et le port du service LoadBalancer est à nouveau autorisé.

2. Pour refuser tout le trafic TCP et UDP entrant en provenance de l'adresse IP source de votre système vers l'adresse IP et le port du service LoadBalancer, créez une règle Pre-DNAT de poids faible nommée `deny-lb-port-80.yaml` dans un éditeur de texte. En utilisant les valeurs de votre aide-mémoire, remplacez `<loadbalancer_IP>` par l'adresse IP publique du service LoadBalancer et `<client_address>` par l'adresse IP publique de l'adresse IP source de votre système.
    ```
    apiVersion: projectcalico.org/v3
    kind: GlobalNetworkPolicy
    metadata:
      name: blacklist
    spec:
      applyOnForward: true
      ingress:
      - action: Deny
        destination:
          nets:
          - <loadbalancer_IP>/32
          ports:
          - 80
        protocol: TCP
        source:
          nets:
          - <client_address>/32
      - action: Deny
        destination:
          nets:
          - <loadbalancer_IP>/32
          ports:
          - 80
        protocol: UDP
        source:
          nets:
          - <client_address>/32
      preDNAT: true
      selector: ibm.role=='worker_public'
      order: 500
      types:
      - Ingress
    ```
    {: codeblock}

3. Appliquez la règle.
    - Linux :

      ```
      calicoctl apply -f filepath/blacklist.yaml
      ```
      {: pre}

    - Windows et OS X :

      ```
      calicoctl apply -f filepath/blacklist.yaml --config=filepath/calicoctl.cfg
      ```
      {: pre}
  L'adresse IP de votre système figure à présent sur liste noire.

4. En utilisant la valeur de votre aide-mémoire, vérifiez sur votre système que vous ne pouvez pas accéder à l'adresse IP du service LoadBalancer car l'adresse IP de votre système est sur liste noire.
    ```
    curl --connect-timeout 10 <loadbalancer_IP>:80
    ```
    {: pre}
    A ce stade, tout le trafic vers les services NodePorts publics est bloqué et tout le trafic vers le service LoadBalancer public est autorisé. Seul le trafic en provenance de l'adresse IP de votre système sur liste noire vers le service LoadBalancer est bloqué.

5. Pour supprimer cette règle de liste noire :

    - Linux :
      ```
      calicoctl delete GlobalNetworkPolicy blacklist
      ```
      {: pre}

    - Windows et OS X :
      ```
      calicoctl delete GlobalNetworkPolicy blacklist --config=filepath/calicoctl.cfg
      ```
      {: pre}

Parfait ! Vous avez réussi à contrôler le trafic dans votre application en utilisant des règles Calico Pre-DNAT pour placer des adresses IP source sur liste blanche ou sur liste noire.

## Etape suivante ?
{: #whats_next}

* En savoir plus sur comment [contrôler le trafic avec des règles réseau](cs_network_policy.html).
* Pour obtenir d'autres exemples de règles réseau Calico utilisées pour contrôler le trafic en provenance et à destination de votre cluster, vous pouvez consulter les pages [Stars Policy Demo ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](https://docs.projectcalico.org/v3.1/getting-started/kubernetes/tutorials/stars-policy/) et [Advanced network policy ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](https://docs.projectcalico.org/v3.1/getting-started/kubernetes/tutorials/advanced-policy).
