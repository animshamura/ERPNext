# ERPNext

# Requirememts: 
- A cluster with MetalLB installed
- A NFS server as a storage class
- NFS client installed in all cluster nodes
  
# Setup Procedure : 
Create a namespace.
```
kubectl create namespace
```
Shift to the erpnext namespace.
```
kubens erpnext
```
Add frappe helm repo.
```
helm repo add frappe https://helm.erpnext.com
```
Install helm chart and create repo.
```
helm install frappe-bench -n erpnext -f custom-values.yaml frappe/erpnext
```
Generate a yaml for createsite.
```
helm template frappe-bench -n erpnext frappe/erpnext -f custom-values.yaml -s templates/job-create-site.yaml > create-new-site-job.yaml
```
Create job resource.
```
kubectl apply -f create-new-site-job.yaml
```
