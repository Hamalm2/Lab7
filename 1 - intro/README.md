## Setting up Helm

*Before you Begin*

Make sure to delete all resources from your cluster from the previous unit. Remember PVC do not get removed when you delete resources manually.

Make sure they are deleted
```
kubectl delete pvc --all
```

## Download and install helm

Run the following commands


download
```
wget https://get.helm.sh/helm-v3.2.4-linux-amd64.tar.gz
```
unpack
```
tar -zxvf helm-v3.2.4-linux-amd64.tar.gz 
```
move the binary
```
sudo mv linux-amd64/helm /usr/local/bin/helm
```

You may need to run the following to make helm executable:
```
sudo chmod +x /usr/local/bin/helm
```

Check your installation by running:
```
helm version
```

add the stable repo
```
helm repo add stable https://charts.helm.sh/stable
```
and update

```
helm repo update
```

---

## Install our first chart

lets install mysql
```
helm install crouchingtiger stable/mysql
```
we should see output like:
```
NAME: crouchingtiger
LAST DEPLOYED: Thu Aug  6 21:18:08 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
crouchingtiger-mysql.default.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default crouchingtiger-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h crouchingtiger-mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following command to route the connection:
    kubectl port-forward svc/crouchingtiger-mysql 3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}

```
Now run the following
```
kubectl get pods
kubectl get svc
kubectl get pvc
```
Nice - no ymls needed and we have a running mysql instance 



Now, lets find and delete it. This will list the names of installed charts.
```
helm ls
```
when we installed we used a name - `crouchingtiger`

we could also use the flag `--generate-name` and well get a random name

delete the install
```
helm delete crouchingtiger
```
___

