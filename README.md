# ERPNext

# Requirememts: 
- A cluster with MetalLB installed
- A NFS server as a storage class
- NFS client installed in all cluster nodes
- Ingress controller in the cluster

  
# Setup Procedure : 
Create a namespace.
```
kubectl create namespace
```
Shift to the erpnext namespace.
If Kubectx hasn't been installed, use ``` sudo snap install kubectx --classic ``` .
```
kubens erpnext
```
Add frappe helm repo.
```
helm repo add frappe https://helm.erpnext.com
```
Install helm chart and create release.
```
helm install frappe-bench -n erpnext -f custom-values.yaml frappe/erpnext
```
Generate an yaml for create-site.
```
helm template frappe-bench -n erpnext frappe/erpnext -f custom-values.yaml -s templates/job-create-site.yaml > create-new-site-job.yaml
```
Apply create-site yaml to create a new site. 
```
kubectl apply -f create-new-site-job.yaml
```
Generate ingress yaml.
```
helm template frappe-bench -n erpnext frappe/erpnext -f custom-values.yaml -s templates/ingress.yaml > ingress.yaml
```
Create ingress resource.
```
kubectl apply -f ingress.yaml
```
Generate backup-site yaml.
```
helm template frappe-bench -n erpnext frappe/erpnext -f custom-values.yaml -s templates/job-backup.yaml > job-backup.yaml
```
Create backup-site resource.
```
kubectl apply -f job-drop-site.yaml
```
Restore the backup applying following command.
```
bench --site [sitename] --force restore [path sql database which file name] --with-private-files[filename] --with-public-files[filename]
```
Generate a drop-site yaml.
```
helm template frappe-bench -n erpnext frappe/erpnext -f custom-values.yaml -s templates/job-drop-site.yaml > job-drop-site.yaml
```
Apply drop-site yaml to archive a site.
```
kubectl apply -f job-drop-site.yaml
```

