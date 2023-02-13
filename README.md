# helm kubernetes python-app & redis deployment

## description

- creating 2 helm charts one for python app and the other for redis from scratch using my own images in dockerhub (ahmednasrhassan/redis - ahmednasrhassan/devops-challenge)

- define python app helm chart to be depnedent on redis chart to install both using one line :rocket: :rocket:

```bash
    helm install python-app-release ./python-app/
```

- testing the app :wink:
    1. create nodeport service (in helm template)
    2. get node ip

    ```bash
         kubectl get nodes -o wide
    ```
    3. curl node-ip:nodeport 

```bash
    ahmednasr@nasr-pc:~/helm/lab2$ helm install python-app-release ./python-app/
    NAME: python-app-release
    LAST DEPLOYED: Mon Feb 13 22:18:39 2023
    NAMESPACE: default
    STATUS: deployed
    REVISION: 1
    ahmednasr@nasr-pc:~/helm/lab2$ k get all
    NAME                                                 READY   STATUS    RESTARTS   AGE
    pod/python-app-release-deployment-7cd44964d4-62xct   1/1     Running   0          10s
    pod/python-app-release-deployment-7cd44964d4-pnk74   1/1     Running   0          10s
    pod/python-app-release-redis-7ff784578f-8wb5p        1/1     Running   0          10s

    NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    service/kubernetes                   ClusterIP   10.96.0.1       <none>        443/TCP          10h
    service/python-app-release-service   NodePort    10.101.30.154   <none>        8000:30080/TCP   10s
    service/redis                        ClusterIP   10.98.138.48    <none>        6379/TCP         10s

    NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/python-app-release-deployment   2/2     2            2           10s
    deployment.apps/python-app-release-redis        1/1     1            1           10s

    NAME                                                       DESIRED   CURRENT   READY   AGE
    replicaset.apps/python-app-release-deployment-7cd44964d4   2         2         2       10s
    replicaset.apps/python-app-release-redis-7ff784578f        1         1         1       10s
    ahmednasr@nasr-pc:~/helm/lab2$ curl 192.168.49.2:30080
    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="utf-8" />
    <link rel="stylesheet" type="text/css" href="/static/css/style.css?v=64d12fd9c6484b319e192082c6c64592">
    <title>Hello, world!</title>
    </head>
    <body>
    <div>
    <p>
    👋 Hi from
    <span class="highlight" id="NASR">NASR</span>
    environment!
    </p>
    <p><span class="counter">Counter:</span> 1</p>
    </div>
    <div>
    <button type="button" onClick="window.location.reload()">Reload</button>
    </div>
    </body>
    </html>
```

## helm QA

## install helm

    ```bash
    curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
    sudo apt-get install apt-transport-https --yes
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
    sudo apt-get update
    sudo apt-get install helm

    ```

1. Add bitnami helm chart repository in the controlplane node.

    ```bash
    controlplane $ helm repo add my-repo https://charts.bitnami.com/bitnami
    ```

2. Deploy the Apache application on the cluster using the apache from the bitnami repository.
Set the release Name to: amaze-surf

    ```bash
    controlplane $ helm install amaze-surf my-repo/apache 
    NAME: amaze-surf
    LAST DEPLOYED: Sun Feb 12 11:07:59 2023
    NAMESPACE: default
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    CHART NAME: apache
    CHART VERSION: 9.2.15
    APP VERSION: 2.4.55

    controlplane $ k get all
    NAME                                    READY   STATUS    RESTARTS   AGE
    pod/amaze-surf-apache-97654464b-rjgjx   0/1     Running   0          9s

    NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
    service/amaze-surf-apache   LoadBalancer   10.107.123.239   <pending>     80:31233/TCP,443:30460/TCP   9s
    service/kubernetes          ClusterIP      10.96.0.1        <none>        443/TCP                      16d

    NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/amaze-surf-apache   0/1     1            0           9s

    NAME                                          DESIRED   CURRENT   READY   AGE
    replicaset.apps/amaze-surf-apache-97654464b   1         1         0       9s
    ```

3. Uninstall the apache chart release  from the cluster.

    ```bash
    controlplane $ helm uninstall amaze-surf 
    release "amaze-surf" uninstalled
    ```

4. install specfic version of nginx 1.22.0 , then update it to specfic 1.23.1 , then rollback

    ```bash
    controlplane $ helm search repo nginx --versions 
    # use chart version crossponds to required app version 
    controlplane $ helm install nginx-release  my-repo/nginx --version 12.0.6
    NAME: nginx-release
    LAST DEPLOYED: Sun Feb 12 11:29:42 2023
    NAMESPACE: default
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    CHART NAME: nginx
    CHART VERSION: 12.0.6
    APP VERSION: 1.22.0

    controlplane $ k get all
    NAME                                READY   STATUS              RESTARTS   AGE
    pod/nginx-release-97fc8d594-cvjrs   0/1     ContainerCreating   0          8s

    NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    service/kubernetes      ClusterIP      10.96.0.1        <none>        443/TCP        16d
    service/nginx-release   LoadBalancer   10.109.111.140   <pending>     80:31549/TCP   8s

    NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx-release   0/1     1            0           8s

    NAME                                      DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx-release-97fc8d594   1         1         0       8s
    ```

    Upgrade

    ```bash
    controlplane $ helm upgrade nginx-release  my-repo/nginx --version 13.2.10
    ```
    
    Output

    ```bash
    Release "nginx-release" has been upgraded. Happy Helming!
    NAME: nginx-release
    LAST DEPLOYED: Sun Feb 12 11:33:36 2023
    NAMESPACE: default
    STATUS: deployed
    REVISION: 2
    TEST SUITE: None
    NOTES:
    CHART NAME: nginx
    CHART VERSION: 13.2.10
    APP VERSION: 1.23.1
    
    ```
    Rollback

    ```bash
    controlplane $ helm rollback nginx-release 1
    Rollback was a success! Happy Helming!
    ```

    History

    ```bash
    
    controlplane $ helm history nginx-release
    REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
    1               Sun Feb 12 11:29:42 2023        superseded      nginx-12.0.6    1.22.0          Install complete
    2               Sun Feb 12 11:33:36 2023        superseded      nginx-13.2.10   1.23.1          Upgrade complete
    3               Sun Feb 12 11:35:49 2023        deployed        nginx-12.0.6    1.22.0          Rollback to 1  

    ```
