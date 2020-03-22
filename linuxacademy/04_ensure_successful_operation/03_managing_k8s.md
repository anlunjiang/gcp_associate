# Managing Kubernetes Engine Resources 

# Resizing cluster 

This can be done by going into the cluster in Kubernetes Engine tab in console and  click edit, yuou can now change the number of instances - this can take a little bit of time 

You can also do it from the CLI:
* `gcloud container clusters resize --size=1 --zone=us-central1-b <name_of_cluster>`

**When you need to make a change to the cluster, you use the gcloud container clusters components**, most other stuff to do with K8s will use the kubectl command

# Kubectl commands
* kubectl get
    * `kubectl get <name_of_resource>` - gets a list of those resources for the cluster
* kubectl describe
    * `kubectl describe <name_of_resource>` - give info on that specific resource (a particular node e.g.)
    * system info, external ip address etc
* kubectl logs - shows logs
* kubectl exec
    * `kubectl exec -it <name_of_pod> -- /bin/bash` - can run bash command on a pod, get an interactive terminal
    * Enables us to fetch data from the pod
* kubectl run 
    * `kubectl run demo --image=nginx --port 80` - create a new deployment, under which the pod is running 

## RECAP of deployments

Deployments have which run under them following their rules of their deployment.yaml and daemonset.yaml:

* Deployment.yaml: Desired State Configuration
    * How many instances of a pod we cant to run on the cluster 
    * K8s ensures that those pods are always up and running
* DaemonSet.yaml: 
    * Ensures a copy of a pod runs on each node in a cluster 
    * Good for monitoring

If you kill just a pod with  
`kubectl delete pod <pod_name>` it would start back up, to delete it you need to delete the deployment, which is a resource:

`kubectl get deployments` - list all deployment resources for this project   
`kubectl delete deployment <name_of_deployment>`

## Summary

* When working with the cluster level - use `gcloud container clusters` components command
* When working with actual K8s resources, use the `kubectl` commands