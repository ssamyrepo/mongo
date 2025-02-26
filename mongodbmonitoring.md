 **"MongoDB Monitoring (Kubernetes): CONQUER Prometheus Exporters"**

### **Step 1: Prerequisites**
1. **Install Helm**:
   - Follow the official Helm installation guide: [Helm Installation](https://helm.sh/docs/intro/install/).

2. **Install Prometheus and Grafana**:
   - Use the `kube-prometheus-stack` Helm chart to install Prometheus and Grafana in your Kubernetes cluster:
     ```bash
     helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
     helm repo update
     helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
     ```

3. **Install `kubectl`**:
   - Ensure `kubectl` is installed and configured to access your Kubernetes cluster: [kubectl Installation](https://kubernetes.io/docs/tasks/tools/).

4. **Clone the Repository**:
   - Clone the repository provided in the video description:
     ```bash
     git clone <repository-url>
     cd <repository-folder>
     ```

---

### **Step 2: Deploy MongoDB**
1. **Create a Namespace**:
   ```bash
   kubectl create namespace mongodb
   ```

2. **Deploy MongoDB**:
   - Apply the MongoDB deployment files from the cloned repository:
     ```bash
     kubectl apply -f mongodb/ -n mongodb
     ```

3. **Verify MongoDB Deployment**:
   ```bash
   kubectl get pods -n mongodb
   ```

---

### **Step 3: Deploy MongoDB Exporter**
1. **Add the Prometheus Community Helm Repository**:
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo update
   ```

2. **Customize the MongoDB Exporter Helm Chart**:
   - Create a `values.yaml` file to override the default values:
     ```yaml
     mongodb:
       uri: "mongodb://admin:password123@mongodb.mongodb.svc.cluster.local:27017"
     serviceMonitor:
       enabled: true
       additionalLabels:
         release: prometheus
     ```

3. **Install the MongoDB Exporter**:
   ```bash
   helm install mongodb-exporter prometheus-community/prometheus-mongodb-exporter -f values.yaml -n mongodb
   ```

4. **Verify the Exporter**:
   ```bash
   kubectl get pods -n mongodb
   ```

5. **Port Forward to Access Exporter Metrics**:
   ```bash
   kubectl port-forward svc/mongodb-exporter 9216:9216 -n mongodb
   ```
   - Access the metrics at `http://localhost:9216/metrics`.

---

### **Step 4: Configure Prometheus to Scrape Metrics**
1. **Verify the ServiceMonitor**:
   - The `serviceMonitor` created by the Helm chart should automatically configure Prometheus to scrape metrics from the exporter.
   ```bash
   kubectl get servicemonitor -n mongodb
   ```

2. **Port Forward to Prometheus**:
   ```bash
   kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090 -n monitoring
   ```
   - Access Prometheus at `http://localhost:9090` and check for the `mongodb_up` metric.

---

### **Step 5: Visualize Metrics in Grafana**
1. **Deploy the Grafana Dashboard**:
   - Apply the Grafana dashboard configuration from the cloned repository:
     ```bash
     kubectl apply -f grafana/ -n monitoring
     ```

2. **Port Forward to Grafana**:
   ```bash
   kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring
   ```
   - Access Grafana at `http://localhost:3000`.

3. **Import the MongoDB Dashboard**:
   - Log in to Grafana (default credentials: `admin/prom-operator`).
   - Import the MongoDB dashboard using the provided configuration.

---

### **Step 6: Clean Up**
1. **Uninstall the MongoDB Exporter**:
   ```bash
   helm uninstall mongodb-exporter -n mongodb
   ```

2. **Delete MongoDB Resources**:
   ```bash
   kubectl delete -f mongodb/ -n mongodb
   ```

3. **Delete the Grafana Dashboard**:
   ```bash
   kubectl delete -f grafana/ -n monitoring
   ```

4. **Delete the Namespace**:
   ```bash
   kubectl delete namespace mongodb
   ```

5. **Remove the Helm Repository**:
   ```bash
   helm repo remove prometheus-community
   ```

