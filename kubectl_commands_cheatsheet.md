# Kubernetes kubectl Command Cheat Sheet

This file contains essential **kubectl commands** used when working with
Kubernetes clusters (for example with Minikube).

------------------------------------------------------------------------

# 1. Cluster Commands

Check cluster information

    kubectl cluster-info

Check nodes in the cluster

    kubectl get nodes

------------------------------------------------------------------------

# 2. View Resources

See running resources

    kubectl get pods
    kubectl get services
    kubectl get deployments
    kubectl get all

More details

    kubectl get pods -o wide

Watch live updates

    kubectl get pods -w

------------------------------------------------------------------------

# 3. Create Resources

Create resources from YAML

    kubectl apply -f deployment.yaml

Create all YAML files in folder

    kubectl apply -f .

Create deployment from command

    kubectl create deployment nginx --image=nginx

------------------------------------------------------------------------

# 4. Describe Resources (Debugging)

    kubectl describe pod POD_NAME
    kubectl describe deployment DEPLOYMENT_NAME
    kubectl describe service SERVICE_NAME

------------------------------------------------------------------------

# 5. View Logs

    kubectl logs POD_NAME

For deployment

    kubectl logs deployment/DEPLOYMENT_NAME

------------------------------------------------------------------------

# 6. Access Container Terminal

    kubectl exec -it POD_NAME -- /bin/sh

or

    kubectl exec -it POD_NAME -- /bin/bash

------------------------------------------------------------------------

# 7. Delete Resources

Delete pod

    kubectl delete pod POD_NAME

Delete deployment

    kubectl delete deployment DEPLOYMENT_NAME

Delete service

    kubectl delete service SERVICE_NAME

Delete from YAML

    kubectl delete -f deployment.yaml

Delete everything

    kubectl delete all --all

------------------------------------------------------------------------

# 8. Update Deployment

Restart deployment

    kubectl rollout restart deployment DEPLOYMENT_NAME

Check rollout status

    kubectl rollout status deployment DEPLOYMENT_NAME

Undo rollout

    kubectl rollout undo deployment DEPLOYMENT_NAME

------------------------------------------------------------------------

# 9. Scale Pods

Increase number of pods

    kubectl scale deployment DEPLOYMENT_NAME --replicas=3

------------------------------------------------------------------------

# 10. Port Forwarding

Access application locally

    kubectl port-forward pod/POD_NAME 3000:3000

or

    kubectl port-forward service/SERVICE_NAME 5000:5000

------------------------------------------------------------------------

# 11. Edit Running Deployment

    kubectl edit deployment DEPLOYMENT_NAME

------------------------------------------------------------------------

# 12. View YAML of Running Resource

    kubectl get deployment DEPLOYMENT_NAME -o yaml

------------------------------------------------------------------------

# 13. Namespace Commands

    kubectl get namespaces
    kubectl create namespace dev
    kubectl delete namespace dev

------------------------------------------------------------------------

# 14. Clean Cluster

Delete everything

    kubectl delete all --all

Delete specific types

    kubectl delete deployments --all
    kubectl delete services --all
    kubectl delete pods --all

------------------------------------------------------------------------

# Most Used Commands (Daily Use)

    kubectl get pods
    kubectl get all
    kubectl apply -f .
    kubectl delete -f .
    kubectl logs POD_NAME
    kubectl describe pod POD_NAME
    kubectl exec -it POD_NAME -- /bin/sh
    kubectl scale deployment
    kubectl rollout restart deployment
    kubectl port-forward

------------------------------------------------------------------------

This cheat sheet covers the most common kubectl commands used for
deploying and managing applications in Kubernetes.
