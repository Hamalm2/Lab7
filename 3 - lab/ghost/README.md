## Ghost


For this part of the lab we will be creating a helm chart for [Ghost](https://github.com/tryghost). Using an existing set of ymls provided in `manifests`

### Start with:

 `helm create ghost`

Your job is to create a new chart called ghost that paramaterises the values in the manifest files. The ones in the files now are the defaults - and also shown below. Use this to help create your `values.yml`

 Note - You will not need all the files generated for you by helm create command (ex: ymls and the test/ dir ). Also make sure the `NOTES.txt` is empty, since it references stuff we wont have. It maybe easier to also clear out the values.yml before starting.




```
imagePullPolicy: "Always"                         # image pull policy

## statefulset:
replicas: 1                                     # The number of replicas
dockerImage: "ghost"                            
dockerTag: "1.24.8-alpine"                      

# Labels
component: "frontend"                       

## Resources
# requests
mem: "200Mi"                                # amout of memory requested
cpu: "200m"                                 # cpu requested
# limits
mem: "200Mi"                                # max memory 
cpu: "200m"                                 # max cpu

# Readiness
initialDelaySeconds: 30                       # time to wait before readiness starts
timeoutSeconds: 5                             # readiness timeout

# Liveness
initialDelaySeconds: 30                       # time to wait before liveness starts
timeoutSeconds: 5                             # liveliness timeout
periodSeconds: 10                             # liveliness period

# Ports 
name: web                                     # name of the port
protocol: TCP                                 # protocol to use
port: 2368                                    # port number
targetPort: 2368                              # target port for the service
```

You are essentially taking the two provided manifests, and making sure they use the paramaterized values from the values.yml file. 

to test, use `kubectl port-forward ghost-0 2368:2368` and navigate to your localhost:2368 to verify. Note - the ghost-0 pod name was found by using `kubectl get pods`

- You should use commands like `helm lint` `debug` and `--dry-run` to validate your new chart


make sure your chart works with new values and as a packaged tarball.


---

- Other helm charts for reference - the [wordpress](https://github.com/bitnami/charts/tree/master/bitnami/wordpress) chart we deployed will be helpful
- https://github.com/muffin87/helm-tutorial (uses older version of helm)
- https://helm.sh/docs/topics/charts/
- https://helm.sh/docs/helm/helm_create/
- https://helm.sh/docs/chart_template_guide/


---
