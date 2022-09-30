https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-kubernetes


- Retrieve your node IPs | kubectl get nodes -o wide
- Get jenkins logs that contains admin password |kubectl logs jenkins-84f5b4bbb7-ktdtx -n jenkins
- Port Forward | kubectl port-forward -n jenkins jenkins-84f5b4bbb7-ktdtx 3000:8080
- Access jenkins pod shell | kubectl exec -i -t -n jenkins jenkins-84f5b4bbb7-ktdtx -c jenkins -- sh -c "clear; (bash || ash || sh)"

kubectl delete namespace jenkins
kubectl create namespace jenkins
kubectl create -f jenkins-pv-pvc.yaml --namespace jenkins
kubectl create -f jenkins-deployment.yaml --namespace jenkins
kubectl create -f jenkins-service.yaml --namespace jenkins