
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

**********************************************************************************************************************************

- Ephemeral volumes: Pod की lifetime तक रहते हैं। उदाहरण: emptyDir, configMap, secret.
- Persistent volumes: Pod के lifecycle से बाहर, cluster-level resource जो डेटा को persist करता है। उपयोग के लिये PV/PVC मॉडल चाहिए।
- CSI (Container Storage Interface): Modern plugin model जिससे cloud और on‑prem storage drivers Kubernetes के साथ integrate करते हैं। हर cloud provider (Azure, AWS, GCP) और कई third‑party vendors CSI drivers देते हैं।

**PersistentVolume (PV) और PersistentVolumeClaim (PVC)**

- PV: Admin द्वारा या dynamically provisioned storage resource। capacity, accessModes, storageClass, reclaimPolicy जैसी जानकारी रखता है।
- PVC: Developer/Pod का request — size, accessModes और optionally storageClass मांगता है। Matching PV से bind होता है या StorageClass के ज़रिये dynamically provision होता है।
YAML — static PV + PVC example (NFS backend):
```
# pv-nfs.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: /export/data
    server: 10.0.0.50
  persistentVolumeReclaimPolicy: Retain
---
# pvc-nfs.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```

- इस example में PVC 10Gi मांगता है; cluster का PV (20Gi, ReadWriteMany) उसके साथ bind होगा।

**StorageClass और Dynamic Provisioning**

- StorageClass बताता है कौन सा provisioner use होगा और किस तरह का disk बनाना है (type, zone, parameters)।
- जब PVC में storageClassName दिया होता है और matching SC उपलब्ध है, cluster dynamically PV create कर देता है।
YAML — Azure Disk StorageClass + PVC example:
```
# sc-azure-disk.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azure-managed-standard
provisioner: disk.csi.azure.com
parameters:
  skuName: Standard_LRS
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
---
# pvc-azure-disk.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-azure
spec:
  storageClassName: azure-managed-standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
```

- WaitForFirstConsumer zone-aware scheduling के लिये अच्छा है — PVC तभी provision होगा जब Pod scheduling zone पता हो।

**Pod में Mount करना — Deployment और StatefulSet का फर्क**

- Deployment के साथ आमतौर पर_SHARED या RWO volumes उपयोग होते हैं; stateful workloads के लिए StatefulSet बेहतर।
- StatefulSet हर replica के लिये अलग PVC बनाता है (volumeClaimTemplates), जिससे stable network id और persistent storage मिलता है।
YAML — StatefulSet with per‑replica PVC:
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web-svc"
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      storageClassName: azure-managed-standard
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi


- हर Pod को अपना dedicated PVC मिलेगा: www-web-0, www-web-1, इत्यादि।

Access Modes 

- **ReadWriteOnce (RWO)**
  - क्या करता है: PV को केवल एक single node पर mount करके read और write दोनों operations किए जा सकते हैं।
  - Typical use case: Single‑node database replica या stateful app जहाँ disk local attach होना चाहिए (e.g., cloud block disk).
    
- ReadOnlyMany (ROX)
  - क्या करता है: PV को कई nodes पर mount किया जा सकता है, लेकिन सिर्फ़ read‑only mode में।
  - Typical use case: Static content distribution जैसे shared config या binaries जहाँ write की ज़रूरत नहीं होती.
    
- ReadWriteMany (RWX)
  - क्या करता है: PV को multiple nodes पर mount करके उन सब से read और write दोनों करने की अनुमति देता है।
  - Typical use case: Shared file storage जैसे CMS uploads, shared caches, या applications जो concurrent writes संभाल सकते हैं; इसके लिये backend को distributed filesystem होना चाहिए (NFS, Gluster, CephFS, or CSI drivers that provide RWX).

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-rwo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: fast-ssd

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-rwx
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  storageClassName: nfs-shared
```
    
- ReclaimPolicy
  - Delete: PVC delete होने पर underlying storage भी delete
  - Retain: data safe रहता है, manual cleanup चाहिए
  
- Common patterns
  - Databases: StatefulSet + RWO + zone-aware disks
  - Shared content: NFS/Gluster/Ceph with RWX
  - Logs/ephemeral scratch: emptyDir या PVC with shorter lifecycle

Troubleshooting और Best Practices
- Use kubectl describe pvc <name> और kubectl get pv to check binding status.
- Pending PVC अक्सर storageClass mismatch, insufficient capacity, या wrong accessModes की वजह से होता है.
- Prefer CSI drivers; deprecated in-tree volume plugins से migrate करें.
- Use volumeBindingMode: WaitForFirstConsumer for multi‑AZ clusters to ensure correct zone provisioning.
- Set appropriate reclaimPolicy depending on whether you want data retained after deletion.
- Encrypt at rest (cloud provider disks) और enable snapshots/backups (VolumeSnapshot CSI).

**Kubernetes में एक PersistentVolume (PV) सामान्यतः एक PersistentVolumeClaim (PVC) के साथ एक‑to‑one तरीके से bind होता है; एक PV एक से ज़्यादा PVCs को सीधे bind नहीं करता.**

जब आप data share करना चाहते हैं

- एक PVC को multiple Pods attach कर सकते हैं — कई Pods वही PVC mount करके data share कर सकते हैं; यह सामान्य pattern है जब आप एक shared volume चाह्ते हैं और access mode अनुमति देता है.
- PV को कई PVCs में बांटना नहीं होता; shared access चाहिए तो अक्सर एक ही PVC बनाकर उसे कई Pods में mount किया जाता है.
  
कब multi‑writer possible है और कैसे

- Access mode जरूरी है: अगर आपको कई nodes से concurrent read/write चाहिए तो underlying storage का support होना चाहिए और PVC में ReadWriteMany (RWX) उपलब्ध होना चाहिए; typical backends: NFS, CephFS, GlusterFS, Azure Files आदि.
- वैकल्पिक तरीके: कुछ CSI drivers cloning/snapshot से multiple PVCs बना सकते हैं (clone से अलग copies मिलती हैं, shared volume नहीं) या vendor‑specific multi‑attach/block features होते हैं पर ये vendor पर depend करते हैं.
- 
छोटा practical example — एक PVC को दो Pods में mount करना
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-pvc
spec:
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs-shared
---
apiVersion: v1
kind: Pod
metadata:
  name: app-a
spec:
  containers:
  - name: app
    image: busybox
    command: ["sleep","3600"]
    volumeMounts:
    - name: shared
      mountPath: /data
  volumes:
  - name: shared
    persistentVolumeClaim:
      claimName: shared-pvc
---
apiVersion: v1
kind: Pod
metadata:
  name: app-b
spec:
  containers:
  - name: app
    image: busybox
    command: ["sleep","3600"]
    volumeMounts:
    - name: shared
      mountPath: /data
  volumes:
  - name: shared
    persistentVolumeClaim:
      claimName: shared-pvc
```

Recommendation / Best practice
- अगर आप shared read/write चाहते हैं तो RWX compatible storage चुनें और application‑level concurrency handling जरूर validate करें.
- अगर per‑pod dedicated storage चाहिए तो हर pod के लिये अलग PVC (StatefulSet का pattern) उपयोग करें.
Sources:

**PVC resize kab possible hai (conditions)**
- StorageClass allowVolumeExpansion = true ya CSI driver expansion support hona chahiye.
- CSI driver / cloud provider ko underlying volume expand karne ki capability honi chahiye; kuch in-tree block volumes limited ya vendor‑specific hoti hain.
- Filesystem type expand hona chahiye (ext4, xfs common supported types). Online expansion possible hai agar driver and filesystem support kare.
- **Shrinking supported nahi hai; aap PVC ko chhota nahi kar sakte.**
- (Static PV special case) Agar PV statically provisioned hai aur StorageClass/driver expansion nahi karta, to admin ko underlying volume manually expand karna padega aur PV spec update karna padega then PVC resize/adjust karna padega.

Kaise resize karte hain — high level steps
- Ensure StorageClass allowVolumeExpansion true और CSI driver support present.
- Edit the PVC resources.request to new size: kubectl patch pvc <name> -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}' या kubectl edit pvc <name> और size badhaye.
- Kubernetes will ask the CSI driver / cloud provider to expand the underlying volume; monitor PVC and PV status with kubectl get pvc और kubectl describe pvc <name>.
- If Pod is running, filesystem may be resized automatically if driver supports online resize; otherwise unmount or restart Pod as required.
