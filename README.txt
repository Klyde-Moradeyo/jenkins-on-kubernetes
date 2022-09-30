https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-kubernetes


-  retrieve your node IPs --- kubectl get nodes -o wide
-  Get jenkins logs that contains admin password --- kubectl logs jenkins-794699f9bc-k2fv4 -n jenkins
- kubectl port-forward -n jenkins jenkins-df8f7db6-v2j9s 3000:8080
- Access jenkins pod shell -- kubectl exec -i -t -n jenkins jenkins-794699f9bc-k2fv4 -c jenkins -- sh -c "clear; (bash || ash || sh)"