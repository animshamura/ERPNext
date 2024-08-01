# ERPNext

## ■ Requirememts: 
- A cluster with MetalLB and Ingress Controller
- A NFS server as a storage class
- NFS client installed in all nodes
  
## ■ Setup Procedure : 

### ■ Step 1:
Create an Orgpod. This will create a ```namespace``` and an ```ingress-class```. 
### ■ Step 2:
Shift to the namespace that's been created by the Orgpod. 
```
kubens erpnext
```
### ■ Step 3:
Add frappe helm repo.
```
helm repo add frappe https://helm.erpnext.com
```
### ■ Step 4:
Before installing helm chart, edit the custom-values.yaml file. Make sure :</br> 
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
### ■ Step 5:
Make ```enabled : true``` and mention the ```ingress-class``` created by the Orgpod in ingress section and generate ingress yaml.
```
helm template frappe-bench -n erpnext frappe/erpnext -f custom-values.yaml -s templates/ingress.yaml > ingress.yaml
```
Create ingress resource.
```
kubectl apply -f ingress.yaml
```
### ■ Step 6:
Make ```enabled : true``` in createsite section and generate an yaml for create-site.
```
helm template frappe-bench -n erpnext frappe/erpnext -f custom-values.yaml -s templates/job-create-site.yaml > create-new-site-job.yaml
```
Apply create-site yaml to create a new site. 
```
kubectl apply -f create-new-site-job.yaml
```
### ■ Step 7:
After having updation through the UI by the erpnext-backend team, following bench commands have to be executed in the Nginx container. </br> 
```
#!/usr/bin/bash
bench --site aeslerp.com install-app custom_erp_app
bench --site aeslerp.com reload-doctype Employee
bench --site aeslerp.com reload-doctype "Job Requisition"
bench --site aeslerp.com reload-doctype "Job Opening"
bench --site aeslerp.com reload-doctype "Workflow Document State"
```


