##  1 - Creating a Chart

lets create a helm chart 
```
helm create chart
```
we should see output like:
```
Creating chart
```
And the following directory structure:

```
chart
|-- Chart.yaml
|-- charts
|-- templates
|   |-- NOTES.txt
|   |-- _helpers.tpl
|   |-- deployment.yaml
|   |-- ingress.yaml
|   |-- hpa.yaml
|   |-- serviceaccount.yaml
|   `-- service.yaml
`-- values.yaml
```

As discussed before, the majority of the stuff that makes a chart work is in the `templates/` directory

This is where the templating happens using [go templates](https://golang.org/pkg/text/template/)


If we open one of the files, specifically `tempates/service.yml` we see:

```
$ cat templates/service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: {{ include "chart.fullname" . }}
  labels:
    {{- include "chart.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "chart.selectorLabels" . | nindent 4 }}

```

When the templating engine runs - this file will look more like a real service definition that we are used to.

Run a dry run so we can see what gets generated.

```
helm install chart --dry-run --debug ./chart
```

In this command we give it the name, some flags for debug/dry run and point it to the directory to install.

In the output we should have seen
```
---
# Source: chart/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: chart
  labels:
    helm.sh/chart: chart-0.1.0
    app.kubernetes.io/name: chart
    app.kubernetes.io/instance: chart
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: chart
    app.kubernetes.io/instance: chart
---
```

Which is what weve seen before, a valid service definition.

We can also add values to our install using `--set`

```
helm install chart --dry-run --debug ./chart --set service.name=lab_chart
```

### Helpers and NOTES


The templates also use partials and functions that were created in the `_helpers.tpl` file. Visit [helm documentation](https://helm.sh/docs/chart_template_guide/getting_started/) to see all the available uses - including flow control.

The `NOTES.txt` file is another file that gets ran through the templating engine, and is printed out after a chart is deployed.

---

## 2 - Install our custom chart

```
helm install chart ./chart --set service.type=NodePort
```

The output of this command is from the `NOTES.txt` file and should show the following:

```
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace dev -o jsonpath="{.spec.ports[0].nodePort}" services chart)
  export NODE_IP=$(kubectl get nodes --namespace dev -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```
running those commands in your terminal should print out a url where you can visit in your browswer and see the deployed application.



## 3 - Adding custom image

Delete the last release : `helm delete chart`

Update the `values.yml` file and change the keys for image to

```
image:
  repository: prydonius/todo
  tag: 1.0.0
  pullPolicy: IfNotPresent
```

Again install the chart like before and get the url to view the application.

## Try with another image

Try doing this with another docker image you can fine online. Note youll have to use a standalone image that doesnt need dependencies.



## Package the chart

run the following:

```
helm package ./chart
```

This will tar up the package and put it in the directory you are in. It should also print out where it was packaged to.  Lets now try to install the package directly using that path to the package.


```
helm install chart /home/CSCC/afarag/helm_lab/chart-0.1.0.tgz --set service.type=NodePort
```

This should behave like before - since all we are doing is pointing to a tar vs a file. If we remember - thats all what helm charts/repos are when they are published to repositories. Its a fancy hosted directory to packaged up helm charts.


## Adding dependencies (OPTIONAL)

Helm allows you to use other charts as dependencies by updating the Chart.yml. lets update it and add a dependency.

read the helm dependency docs [here](https://helm.sh/docs/helm/helm_dependency/)

```
vim chart/Chart.yaml
```

lets add a dependency, in the file add the following

```
dependencies:
  - name: mysql
    version: 1.6.7
    repository: https://charts.helm.sh/stable
    tags:
      - mysql
```

then update, repackage and install:
```
helm dep update ./chart
helm package ./chart
helm install chart /home/CSCC/afarag/helm_lab/chart-0.1.0.tgz --set service.type=NodePort
```

watch using `kubectl get all` and see that both our container and the resources from the mysql chart are deployed.

These are the building blocks for building applications using helm/k8. We template out the resources with default values, then connect them with other charts - rather than trying to manage it all together.
