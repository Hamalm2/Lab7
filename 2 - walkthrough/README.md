## Wordpress example

In lab3 we made a wordpress application and deployed it. It took 6 yml files. We could have put them all in one, but regardless there was alot of config.

in a second terminal run
```
watch kubectl get all
```

lets use a chart to install wordpress

if we visit the [wordpress chart](https://github.com/helm/charts/tree/master/stable/wordpress) repo. It refers us to use a different repository [here](https://github.com/bitnami/charts/tree/master/bitnami/wordpress)

lets add it and install-
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

```

```
helm install my-wordpress bitnami/wordpress
```

should see the following :

```
NAME: wordpress
LAST DEPLOYED: Thu Aug  2 21:58:27 2020
NAMESPACE: dev
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    wordpress.dev.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace dev -w wordpress'

   export SERVICE_IP=$(kubectl get svc --namespace dev wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace dev wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)

```


cool, we should see the pods coming up in the second terminal. lets see if we can access it

```
kubectl get svc
```
hmm looks like the wordpress default value is configured to use a LoadBalancer by default, which is not supported with minikube.
```
NAME                TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
wordpress           LoadBalancer   10.103.73.102    <pending>     80:32283/TCP,443:30676/TCP   2m54s
wordpress-mariadb   ClusterIP      10.109.117.233   <none>        3306/TCP                     2m54s
```

Delete the release & the PVC and then lets configure the values so we can use a nodeport.

Remember we can either use configuration from a yaml file or pass them in as parameters.

---
## Using a file ->

Download the default values so we can edit them
```
wget https://raw.githubusercontent.com/bitnami/charts/master/bitnami/wordpress/values.yaml
```

Add any non-empty strings you like for wordpresspassword, mariadb.auth.rootPassword, mariadb.auth.password

update the values in service.type to NodePort, and set the nodePorts.http to "30000" and nodePorts.https to "30001"

if we want to validate our values before we run we use the `dry-run` command

```
helm install my-wordpress bitnami/wordpress --dry-run -f values.yaml
```

this is useful when debugging installation failures. for more see [here](https://helm.sh/docs/chart_template_guide/debugging/)

```
helm install my-wordpress bitnami/wordpress -f values.yaml
```
---
## Using params -> 

```
helm install my-wordpress bitnami/wordpress --set service.type=NodePort --set service.nodePorts.http=30000 --set service.nodePorts.https=30001
```

---

*notice* - 
helm will use values from the provided file or the command line arguments *OVER*  the ones in the default chart.

lets verify this.
```
# this will show user supplied
helm get values my-wordpress
helm get values my-wordpress --all
```

---
once the pods are both up, use the minikube service command to expose the port
```
minikube service my-wordpress
```

you should be able to visit `https://172.17.0.3:30001/` or `http://172.17.0.3:30000/` and see the application running!

---

As we learned, helm takes a template from a chart along with user supplied config + default configs and creates the manifest files for kubernetes to use.

lets see what we get with our wordpress installation
```
helm get manifest my-wordpress > wp.yaml
```

we can now see exactly what is being rendered for kubernetes to consume in the wp.yaml file. This is very useful when debugging applications

you *technically* can use this manifest directly. try it out. uninstall the release and pvc, then run `kubectl apply -f wp.yaml`

when done delete this using
```
kubectl delete -f wp.yaml
kubectl delete pvc --all
```
---

## helm versioning

with a clean cluster, install a new chart:  the wordpress available from stable repo

```
helm install stable-wordpress stable/wordpress
```

lets perform an upgrade by changing the value of image.tag with the --set operator
```
helm upgrade stable-wordpress stable/wordpress --set image.tag=latest
```
We should see the new pod starting and the old one rolling off. 

to validate use describe
```
# use this to get the pod
kubectl get pods
kubectl describe pod stable-wordpress-698558bdd4-gfbpd | grep image
```
we should see our updated image being pulled
---

Lets roll back to the earlier version, as I dont like being on latest.

```
helm list

stable-wordpress	dev      	2       	2020-08-02 23:31:34.720113505 -0400 EDT	deployed	wordpress-9.4.2	5.4.2      
```

here we see that there are two revisions. lets roll back to the first

```
helm rollback stable-wordpress 1
Rollback was a success! Happy Helming!
```

Again we should see the wordpress prod cycling, and after it is up we can verify the image indeed changed, this time using the values command

```
helm get values stable-wordpress --all
```

we should see our original image being used. cool, weve successfully rolled back our application.

---


In this example we showed how useful using helm can be when deploying applications on kubernetes.
