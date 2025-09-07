
# Volumes in Docker 

When Docker is installed, it organizes data within the **/var/lib/docker** directory. This folder contains several subdirectories such as aufs, containers, images, and volumes. Each subdirectory serves a specific role in Docker’s architecture:

**containers**: Stores all files related to running containers.

**images**: Contains stored images.

**volumes**: Holds data for persistent storage created by containers.

A Docker image consists of several read-only layers:

**Base Layer**: The Ubuntu operating system.

**Packages Layer**: APT packages installed on top of Ubuntu.

**Dependencies Layer**: Python packages such as Flask.

**Source Code Layer**: Your application code included in the image.

**Entry Point Layer**: The layer that sets the container’s entry point.

* Running a container from this image creates a new writable layer on top, which stores changes such as logs, temporary files, or modifications made during runtime. 
* This mechanism is known as copy-on-write. 
* Even if you modify a read-only file from the image, Docker creates a separate copy in the writable layer before applying the changes.

<img width="528" height="330" alt="image" src="https://github.com/user-attachments/assets/aafb7ea6-7b70-48ec-9a65-f35b3669855d" />

**Remember, when you remove a container, its writable layer and any associated changes will be lost. The original image remains unchanged unless it is rebuilt.**

**Persisting Data with Volumes**

For data persistence outside the container’s ephemeral writable layer—such as database storage—use Docker volumes.

**Creating and Using Volumes**

First, create a volume:
```
docker volume create data_volume
```

Then, mount the created volume when launching a container. For example, to store MySQL data in the volume, run:
```
docker run -v data_volume:/var/lib/mysql mysql
```

If a specified volume does not exist, Docker automatically creates it. To inspect the volumes, you can list the contents of **/var/lib/docker/volumes**.

**Bind Mounts**
Alternatively, if you prefer using external storage (for instance, storing database files in /data/mysql on your host), you can use bind mounts:
```
docker run -v /data/mysql:/var/lib/mysql mysql
```

# Volumes in Kubernetes

In simple words **volume** specifies from where the storage is given, in this case it is a hostPath or the underlying storage. Whereas the **volumeMount** option specifies how the volume should be mounted inside the container . 

```
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
    - image: alpine
      name: alpine
      command: ["/bin/sh", "-c"]
      args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
      volumeMounts:
        - mountPath: /opt
          name: data-volume
  volumes:
    - name: data-volume
      hostPath:
        path: /data
        type: Directory
```

You can also use AWS block storage as below 
```
volumes:
  - name: data-volume
    awsElasticBlockStore:
      volumeID: <volume-id>
      fsType: ext4
```

## Persistent Volumes

* Persistent volumes use centralized storage management.
* Administrators can create a large pool of storage, and users can request specific portions from that pool using Persistent Volume Claims (PVCs).

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /pv/log
```

```
kubectl create -f pv-definition.yaml

kubectl get persistentvolume
```

## Persistent Volume Claims

Using PVC you can claim the required storage from the underlying PV

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

```
kubectl create -f pvc-definition.yaml

kubectl get persistentvolumeclaim
```

