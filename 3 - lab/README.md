# Infrastructure Automation Helm Lab

## Introduction

Creating our own custom charts with using helm.


## Objectives

We will use helm to create custom charts for a couple applications. They are simple, but you will gain experience with helm and its core ideas/

---
## Getting Started:


Make sure to delete all resources from your cluster from the previous unit. Remember PVC do not get removed when you delete resources manually.

Make sure they are deleted
```
kubectl delete pvc --all
```



**Make sure you have helm installed from part 1 of this repo**

---

## Understanding the Starter Code
There are two parts to this lab
1. create_chart
2. ghost


Within each directory, there is a readme with the excerise. Follow the directions, and save your work as it will be turned in.


---

## Completing the Assignment

Go through the excerise in the `create_chart` directory and complete the assingment in the `ghost`


## Submitting Your Work

1-  Publish your repository as a private repo, and ensure that you have pushed the latest version

2-  Submit the assignment in Blackboard with the link to your repo



---
## Resources

- Other helm charts for reference - the [wordpress](https://github.com/bitnami/charts/tree/master/bitnami/wordpress) chart we deployed will be helpful
- https://github.com/muffin87/helm-tutorial (uses older version of helm)
- https://helm.sh/docs/topics/charts/
- https://helm.sh/docs/helm/helm_create/
- https://helm.sh/docs/chart_template_guide/
