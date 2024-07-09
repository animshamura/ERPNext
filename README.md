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
