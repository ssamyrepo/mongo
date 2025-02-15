### **How MongoDB Stores Data in a Kubernetes (K8s) Cluster?**
When **MongoDB is deployed on a Kubernetes cluster**, data storage depends on **Persistent Volumes (PVs) and Persistent Volume Claims (PVCs)**. Kubernetes provides **persistent storage to ensure MongoDB data survives pod restarts, failures, and rescheduling**.

---

## **1. Storage Architecture of MongoDB on Kubernetes**
MongoDB running in Kubernetes requires **persistent storage** to store database files across pod restarts.

### **How Data is Stored in Kubernetes**
| **Storage Layer**   | **Component in Kubernetes** | **MongoDB Equivalent** |
|---------------------|----------------------------|------------------------|
| **Physical Storage** | AWS EBS, GCP PD, Azure Disk, NFS, Ceph, Longhorn | Disk, SSD |
| **Persistent Volume (PV)** | Defines storage resource | Physical database storage |
| **Persistent Volume Claim (PVC)** | Requests PV resources | MongoDB’s `/data/db` mount |
| **StatefulSet Pod** | Runs MongoDB container | MongoDB instance |

#### **Storage Flow in MongoDB on K8s**
1. MongoDB pod is deployed as a **StatefulSet**.
2. A **Persistent Volume Claim (PVC)** is created for **each MongoDB replica**.
3. The PVC requests storage from **Persistent Volumes (PVs)** (e.g., AWS EBS, Azure Disks).
4. The **MongoDB container mounts the PVC to `/data/db`**.
5. Data is **persisted** in PVs, ensuring it survives pod restarts.

---

## **2. Key Components for MongoDB Storage in Kubernetes**
### **A. Persistent Volume (PV)**
PV defines a **physical storage resource** that MongoDB pods will use.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "standard"
  hostPath:
    path: "/mnt/data"
```

---

### **B. Persistent Volume Claim (PVC)**
PVC requests storage from PV for MongoDB.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: "standard"
```

---

### **C. StatefulSet for MongoDB**
A **StatefulSet** ensures each MongoDB pod gets a unique, persistent storage.
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  replicas: 3
  serviceName: "mongo"
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:latest
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongo-storage
          mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: mongo-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

---

## **3. How Data Consistency is Maintained in MongoDB on Kubernetes**
MongoDB uses **Replica Sets** to keep data consistent across multiple pods.

### **Replication Process**
- **Primary Pod** handles all writes.
- **Secondary Pods** replicate data from primary.
- If the primary pod crashes, a **new primary is elected** automatically.
- Persistent storage ensures data survives even if the pod is rescheduled.

### **Ensuring High Availability**
- **Use 3 MongoDB replicas in a StatefulSet**.
- **Distribute pods across multiple Kubernetes nodes**.
- **Enable Read and Write Concerns** for data consistency:
  ```yaml
  writeConcern: { w: "majority" }
  readConcern: "majority"
  ```

---

## **4. What Happens If a Pod Fails?**
- **Pod restart:** It will mount the same PVC and continue serving data.
- **Node failure:** MongoDB StatefulSet ensures a **new pod is created on another node**.
- **Storage failure:** If the underlying PV fails, **backup and restore mechanisms (e.g., MongoDB snapshots, Kubernetes Velero)** can recover the data.

---

## **5. Best Practices for Running MongoDB on Kubernetes**
1. **Use StatefulSets** to ensure persistent storage and stable pod identities.
2. **Use Persistent Volumes (PVs) backed by cloud block storage** (AWS EBS, Azure Disk, GCP PD).
3. **Set proper Read and Write Concerns** for data consistency.
4. **Distribute MongoDB pods across multiple nodes** to avoid a single point of failure.
5. **Enable Kubernetes Liveness & Readiness Probes** to monitor pod health.
6. **Schedule Backups using MongoDB Backup Tools or Velero**.
7. **Use Storage Classes** to dynamically provision volumes for MongoDB.

---

### **Summary**
✅ MongoDB running in **Kubernetes uses PVs and PVCs to persist data**.  
✅ **StatefulSets** manage MongoDB nodes, ensuring high availability.  
✅ **Replica Sets** keep data consistent across pods.  
✅ **If a pod fails, data remains intact via PVCs and automated failover**.

Would you like a **Helm chart for deploying MongoDB in Kubernetes**? 🚀
