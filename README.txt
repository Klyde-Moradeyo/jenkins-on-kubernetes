https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-kubernetes


-  retrieve your node IPs --- kubectl get nodes -o wide
-  Get jenkins logs that contains admin password --- kubectl logs jenkins-6dc59b988b-sqhlt -n jenkins
- kubectl port-forward -n jenkins jenkins-6dc59b988b-sqhlt 3000:8080
- Access jenkins pod shell -- kubectl exec -i -t -n jenkins jenkins-6dc59b988b-sqhlt -c jenkins -- sh -c "clear; (bash || ash || sh)"

kubectl delete namespace jenkins
kubectl create namespace jenkins
kubectl create -f jenkins-pv-pvc.yaml --namespace jenkins
kubectl create -f jenkins-deployment.yaml --namespace jenkins
kubectl create -f jenkins-service.yaml --namespace jenkins