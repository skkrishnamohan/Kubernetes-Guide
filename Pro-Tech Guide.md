## **Introduction**

The Kubernetes Pro-Tech Guide is a curated set of troubleshooting steps, one-liner commands, and automation scripts for resolving common issues encountered in Kubernetes clusters. This document is especially tailored for:

- **DevOps professionals working on AKS private clusters**.
- Teams managing **.NET microservices** or any application workloads.
- Professionals handling **Kubernetes networking, storage, and resource issues**.

With this guide, you can address the most frequent problems, automate repetitive tasks, and efficiently debug your Kubernetes workloads.

---

## **Troubleshooting Scenarios**

### **1. Fix Expired Kubernetes Certificates**

Steps to fix expired Kubernetes certificates:

1. **Check Certificate Expiry:**
   ```bash
   openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -enddate
   ```

2. **Renew Certificates:**
   ```bash
   kubeadm certs renew all
   ```

3. **Restart Kubernetes Components:**
   ```bash
   systemctl restart kubelet
   ```

4. **Verify Component Status:**
   ```bash
   kubectl get componentstatuses
   ```

---

### **2. Find Pods Scheduled on Master Nodes**

1. **List Pods on Master Nodes:**
   ```bash
   kubectl get pods -o wide --all-namespaces | grep master
   ```

2. **Verify Master Node Taints:**
   ```bash
   kubectl describe node <master-node-name> | grep Taints
   ```

---

### **3. Memory Scheduling Issues**

1. **Check Resource Requests and Limits:**
   ```bash
   kubectl describe pod <pod-name> | grep -A 5 "Resources"
   ```

2. **Check Node Allocatable Memory:**
   ```bash
   kubectl describe node <node-name> | grep Allocatable
   ```

3. **Solutions:**
   - Reduce resource requests in the pod spec.
   - Add nodes to the cluster if memory is insufficient.
   - Scale down non-critical workloads temporarily.

---

### **4. Common Pod Statuses and Fixes**

#### **Pending**
- **Cause:** No available nodes with sufficient resources.
- **Fix:** Add nodes or adjust resource requests.
  ```bash
  kubectl describe pod <pod-name>
  ```

#### **CrashLoopBackOff**
- **Cause:** Application crashes repeatedly.
- **Fix:** Check logs for the root cause.
  ```bash
  kubectl logs <pod-name>
  ```

#### **ImagePullBackOff**
- **Cause:** Docker image cannot be pulled.
- **Fix:** Verify image name and registry credentials.
  ```bash
  kubectl describe pod <pod-name>
  ```

#### **Completed**
- **Cause:** The pod completed its task successfully (e.g., a job).
- **Fix:** None required unless the job should not terminate.

#### **Unknown**
- **Cause:** Node issues, such as connectivity problems.
- **Fix:** Check node status and logs.
  ```bash
  kubectl describe node <node-name>
  ```

#### **Terminating**
- **Cause:** Finalizers or stuck pod deletion process.
- **Fix:** Remove finalizers manually:
  ```bash
  kubectl patch pod <pod-name> -p '{"metadata":{"finalizers":[]}}'
  ```

---

### **5. Service Not Accessible**

1. **Verify Service Configuration:**
   ```bash
   kubectl describe svc <service-name>
   ```

2. **Check Endpoints:**
   ```bash
   kubectl get endpoints <service-name>
   ```

3. **Debug NodePort Services:**
   ```bash
   curl http://<node-ip>:<node-port>
   ```

4. **Check DNS Resolution:**
   ```bash
   kubectl exec -it <pod-name> -- nslookup <service-name>
   ```

---

### **6. PersistentVolume Not Bound**

1. **Verify PVC and PV Details:**
   ```bash
   kubectl describe pvc <pvc-name>
   kubectl describe pv <pv-name>
   ```

2. **Check Storage Classes:**
   ```bash
   kubectl get sc
   ```

3. **Fix Mismatched Claims:**
   - Delete and recreate PVCs or PVs with matching specs.

---

### **7. Ingress Not Routing Requests**

1. **Verify Ingress Configurations:**
   ```bash
   kubectl describe ingress <ingress-name>
   ```

2. **Check Backend Services:**
   ```bash
   kubectl get svc
   ```

3. **Debug Certificate Issues:**
   ```bash
   kubectl describe secret <secret-name>
   ```

---

### **8. Calico Networking Issues**

1. **Check Calico Pod Logs:**
   ```bash
   kubectl logs -n kube-system -l k8s-app=calico-node
   ```

2. **Verify Pod CIDR Configuration:**
   ```bash
   kubectl cluster-info dump | grep -m 1 cluster-cidr
   ```

3. **Restart Calico Pods:**
   ```bash
   kubectl delete pods -n kube-system -l k8s-app=calico-node
   ```

---

## **One-Liner Troubleshooting Commands**

1. **Get all pods with detailed information:**
   ```bash
   kubectl get pods -o wide --all-namespaces
   ```

2. **List nodes and their taints:**
   ```bash
   kubectl get nodes -o json | jq '.items[].spec | {name: .metadata.name, taints: .taints}'
   ```

3. **Debug DNS issues in a pod:**
   ```bash
   kubectl exec -it <pod-name> -- nslookup <service-name>
   ```

4. **Delete all failed pods in a namespace:**
   ```bash
   kubectl get pods --field-selector=status.phase=Failed -o name | xargs kubectl delete
   ```

5. **Restart all pods in a deployment:**
   ```bash
   kubectl rollout restart deployment <deployment-name>
   ```

---

## **Automation Scripts**

### **Script to Delete All Failed Pods**
```bash
#!/bin/bash
kubectl get pods --all-namespaces --field-selector=status.phase=Failed -o name | xargs kubectl delete
```

### **Script to Collect Logs from All Pods in a Namespace**
```bash
#!/bin/bash
NAMESPACE=$1
OUTPUT_DIR=$2

mkdir -p $OUTPUT_DIR

for POD in $(kubectl get pods -n $NAMESPACE -o jsonpath="{.items[*].metadata.name}"); do
  kubectl logs $POD -n $NAMESPACE > $OUTPUT_DIR/$POD.log
done
```

**Usage:**
```bash
./collect-logs.sh <namespace> <output-directory>
```

### **Script to Restart All Pods in a Namespace**
```bash
#!/bin/bash
NAMESPACE=$1

kubectl delete pods --all -n $NAMESPACE
```

**Usage:**
```bash
./restart-pods.sh <namespace>
```

---

## **Additional Topics**

### **TLS Certificates with Let's Encrypt**
1. **Generate Certificates with Certbot:**
   ```bash
   certbot certonly --standalone -d <domain>
   ```

2. **Create Kubernetes Secrets for TLS:**
   ```bash
   kubectl create secret tls <secret-name> --cert=/path/to/fullchain.pem --key=/path/to/privkey.pem
   ```

---

### **Common HTTP/HTTPS Errors in Pods**

#### **HTTP 404 Not Found**
- **Cause:** Service not pointing to the correct pod.
- **Fix:**
  ```bash
  kubectl describe service <service-name>
  ```

#### **HTTP 500 Internal Server Error**
- **Cause:** Application error in the pod.
- **Fix:**
  ```bash
  kubectl logs <pod-name>
  ```

#### **HTTPS Certificate Error**
- **Cause:** Missing or invalid TLS certificates.
- **Fix:**
  ```bash
  kubectl describe secret <secret-name>
  ```

---

## **Real-World Examples**

### **Example 1: Fixing Calico Networking Issues**

1. **Check Calico Node Logs:**
   ```bash
   kubectl logs -n kube-system -l k8s-app=calico-node
   ```

2. **Verify Pod CIDR Configuration:**
   ```bash
   kubectl cluster-info dump | grep -m 1 cluster-cidr
   ```

3. **Restart Calico Pods:**
   ```bash
   kubectl delete pods -n kube-system -l k8s-app=calico-node
   ```

### **Example 2: Debugging API Call Failures**

1. **Check Pod Logs:**
   ```bash
   kubectl logs <pod-name>
   ```

2. **Verify Service and Endpoints:**
   ```bash
   kubectl describe svc <service-name>
   kubectl get endpoints <service-name>
   ```

3. **Test Connectivity Using Curl:**
   ```bash
   kubectl exec -it <pod-name> -- curl -v <service-url>
   ```

---

## **Conclusion**

This Kubernetes Pro-Tech Guide provides a ready reference for troubleshooting, automation, and improving cluster stability. Save this guide to your GitHub repository for future use, and feel free to contribute additional scripts or scenarios to enhance its utility.
