# Overview
- **Jenkins on Kubernetes with Persistent Storage:** This repository is for setting up Jenkins on Kubernetes using Docker Desktop for Windows with WSL 2, and configuring it persistent storage. 

This deployment is designed for local dev enviornments and not production workloads.

# Table of Contents
1. [Overview](#overview)
2. [Jenkins on Kubernetes with Persistent Storage](#jenkins-on-kubernetes-with-persistent-storage)
3. [Prerequisites](#prerequisites)
4. [Useful Commands](#useful-commands)
5. [Docker Desktop for Windows Volume Mounts](#docker-desktop-for-windows-volume-mounts)
5. [Docker Volume mount: Persisting Data when using K8 in Docker Desktop](#docker-volume-mount-persisting-data-when-using-k8-in-docker-desktop)
6. [Host Path: Persisting data when using K8 in Docker Desktop using Docker](#host-path-persisting-data-when-using-k8-in-docker-desktop-using-docker)
7. [Further Reading](#further-reading)

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

# Docker Desktop for Windows Volume Mounts

When using Docker Desktop for Windows with Kubernetes, it's crucial to understand how volume mounts are handled.

**Mounting Volumes**:

To mount volumes, the path structure should be as follows:

```bash
/run/desktop/mnt/host/c/PATH/TO/FILE
```

For instance, if you wish to mount the volume located at C:\users\klyde\dev\jenkins in Windows, the path you would use in your Kubernetes configuration would be:
```
/run/desktop/mnt/host/c/users/klyde/dev/jenkins
```

**Why this Path?** 
This path structure is necessary due to Docker Daemon's handling of cross-distro mounts. Docker Daemon uses /mnt/wsl as the mount point for these mounts, which it then maps to its own /run/desktop/mnt/host/wsl directory.

# Docker Volume mount: Persisting data when using K8 in Docker Desktop

### Overview
While suitable for local development, this approach isn't optimal for production environments requiring distributed, multi-node storage.
Using Docker volumes in a local Kubernetes setup on Docker Desktop within WSL ensures data persistence across restarts. 

### Steps
1. In a WSL 2 terminal, create a Docker volume named `jenkins-data`:

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

# Host Path: Persisting data when using K8 in Docker Desktop using Docker 
This explains how to deploy Jenkins on Kubernetes using `hostPath` volumes for storing Jenkins data. 

By using host path, you are able to view the contents of the `/var/jenkins_home` in your Windows file explorer.

## Commands
- Because we are using host paths, using `jenkins-pv-pvc.yaml` to create a peristent volume is not neccessary.
```
kubectl delete -f jenkins-deployment.yaml
kubectl delete -f jenkins-service.yaml
kubectl create -f jenkins-deployment.yaml
kubectl create -f jenkins-service.yaml
```

## Deployment

The deployment uses the official Jenkins LTS Docker image. 

Replace `/mnt/c/my-directory` with the actual directory on your WSL filesystem that you want to use for storing Jenkins data.

Here's the Kubernetes YAML for the deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: jenkins 
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: jenkins-home
          mountPath: /var/jenkins_home
      volumes:
      - name: jenkins-home
        hostPath:
          # directory location on host (in WSL format)
          path: /run/desktop/mnt/host/s/development/jenkins-data
          # this field is optional
          type: DirectoryOrCreate
```

## Notes

- The `hostPath` volume in the Deployment directly mounts a directory from your host machine into the Pod, effectively bypassing the need for a PersistentVolume (PV) or a PersistentVolumeClaim (PVC). As such, a PVC is not required in this setup.

- Please note that using `hostPath` volumes can have implications. In particular, if you're running a multi-node Kubernetes cluster, keep in mind that `hostPath` volumes are tied to a specific node and will not be accessible if the pod is scheduled on a different node.

- The directory specified in `hostPath` should be in the format of the host machine's filesystem. Since you're running Kubernetes inside Docker Desktop which is running inside WSL2, the `hostPath` should be in the format of the WSL2 filesystem.


**For my own notes :)** 
- Login details:
  - user: admin 
  - pass: admin

# Further Reading
- [How to install jenkins on kubernetes](https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-kubernetes)
- [Kubernetes Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
- [Kubernetes Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Docker Volumes](https://docs.docker.com/storage/volumes/)