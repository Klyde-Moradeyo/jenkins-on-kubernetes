# Overview
- **Jenkins on Kubernetes with Persistent Storage:** This repository is for setting up Jenkins on Kubernetes using Docker Desktop for Windows with WSL 2, and configuring it to use a Docker volume for persistent storage.

# Table of Contents
1. [Jenkins on Kubernetes with Persistent Storage](#jenkins-on-kubernetes-with-persistent-storage)
2. [Prerequisites](#prerequisites)
3. [Useful Commands](#useful-commands)
4. [Persisting Data when using K8 in Docker Desktop](#persisting-data-when-using-k8-in-docker-desktop)
5. [Further Reading](#further-reading)

# Prerequisites
- Docker Desktop for Windows with WSL 2 integration enabled
- Kubernetes enabled on Docker Desktop
- kubectl command-line tool installed

## Useful Commands
- The jenkins pod when first initially start will contain the token for first time access
```
 kubectl logs jenkins-84f5b4bbb7-zwgjx -n jenkins # Get Pods Logs
```

- Retrieve your node IPs
```
 kubectl get nodes -o wide
```

- Port Forward Jenkins pod
```
 kubectl port-forward -n jenkins jenkins-84f5b4bbb7-zwgjx 3000:8080
```
- Access pod shell
```
 kubectl exec -i -t -n jenkins jenkins-84f5b4bbb7-zwgjx -c jenkins -- sh -c "clear; (bash || ash || sh)"
```

- Commands used to reset the deployment
```
 kubectl delete -f .
 kubectl delete namespace jenkins
 kubectl create namespace jenkins
 kubectl apply -f .
 
```



# Persisting data when using K8 in Docker Desktop

### Overview
While suitable for local development, this approach isn't optimal for production environments requiring distributed, multi-node storage.
Using Docker volumes in a local Kubernetes setup on Docker Desktop within WSL ensures data persistence across restarts. 


1. In a WSL 2 terminal, create a Docker volume named `jenkins-data`:

### Steps

```bash
    docker volume create jenkins-data
```


2. Create a PersistentVolume and PersistentVolumeClaim

Create a file named jenkins-pv-pvc.yaml with the following content:

```
    # PersistentVolume for Jenkins 
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: jenkins-pv
      namespace: jenkins
    spec:
      capacity:
        storage: 10Gi
      volumeMode: Filesystem
      accessModes:
        - ReadWriteOnce
      persistentVolumeReclaimPolicy: Retain
      storageClassName: local-storage
      hostPath:
        path: /var/lib/docker/volumes/jenkins-data/_data
    ---
    # PersistentVolumeClaim for Jenkins
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: jenkins-pvc
      namespace: jenkins    # PV are not scoped to any namespace, but pvc is associated with the namespace
    spec:
      storageClassName: local-storage
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
```

# Further Reading
- [How to install jenkins on kubernetes](https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-kubernetes)
- [Kubernetes Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
- [Kubernetes Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Docker Volumes](https://docs.docker.com/storage/volumes/)