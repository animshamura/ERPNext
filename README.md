# ERPNext

## ☐ Requirememts: 
- A cluster with MetalLB installed (<a href="https://github.com/nasirnjs/kubernetes/blob/main/k8s-cluster-setup/k8s-1.29-cluster-setup-ubuntu-22.md">Cluster Setup</a> and <a href="https://github.com/nasirnjs/kubernetes/blob/main/k8s-cluster-setup/metallb-l2-configuration.md">MetalLB Installation</a>)
- A NFS server as a storage class ( <a href="https://github.com/nasirnjs/kubernetes/blob/main/k8s-cluster-setup/dynamic-nfs-provisioning_k8s.md">Setup Dynamic NFS </a>)
- NFS client installed in all cluster nodes (<a href="https://bluexp.netapp.com/blog/azure-anf-blg-linux-nfs-server-how-to-set-up-server-and-client">NFS Client Setup</a>)
- Ingress controller in the cluster (<a href="https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx">Nginx Controller</a>)

  
## ☐ Setup Procedure : 

### Step 1:
Create an Orgpod. This will create a namespace and an ingress-class-name. </br> </br>
### Step 2:
Shift to the namespace that's been created by the Orgpod. 
```
kubens erpnext
```
### Step 3:
Add frappe helm repo.
```
helm repo add frappe https://helm.erpnext.com
```
### Step 4:
Before installing helm chart, edit the custom-values.yaml file.</br> 
Make sure :</br> 
✓ Proper image tag has been mentioned. </br> 
✓ Keep ```enabled : false``` in ingress, createsite, dropsite, backup and migrate.   
✓ Comment out custom_erp_app. 
```
installApps:
    - "frappe"
    - "erpnext"
    - "hrms"
    # - "custom_erp_app"
```
Install helm chart and create release.
```
helm install frappe-bench -n erpnext -f custom-values.yaml frappe/erpnext
```
### Step 5:
Make ```enabled : true``` in ingress and generate ingress yaml.
```
helm template frappe-bench -n erpnext frappe/erpnext -f custom-values.yaml -s templates/ingress.yaml > ingress.yaml
```
Create ingress resource.
```
kubectl apply -f ingress.yaml
```
### Step 6:
Make ```enabled : true``` in createsite and generate an yaml for create-site.
```
helm template frappe-bench -n erpnext frappe/erpnext -f custom-values.yaml -s templates/job-create-site.yaml > create-new-site-job.yaml
```
Apply create-site yaml to create a new site. 
```
kubectl apply -f create-new-site-job.yaml
```
### Step 7:
After having updation through the UI by the erpnext-backend team.</br> 
Following bench commands have to be executed in the Nginx container. </br> 
```
#!/usr/bin/bash
bench --site aeslerp.com install-app custom_erp_app
bench --site aeslerp.com reload-doctype Employee
bench --site aeslerp.com reload-doctype "Job Requisition"
bench --site aeslerp.com reload-doctype "Job Opening"
bench --site aeslerp.com reload-doctype "Workflow Document State"
```


