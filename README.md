# ERPNext

## ☐ Requirememts: 
- A cluster with MetalLB installed (<a href="https://github.com/nasirnjs/kubernetes/blob/main/k8s-cluster-setup/k8s-1.29-cluster-setup-ubuntu-22.md">Cluster Setup</a> and <a href="https://github.com/nasirnjs/kubernetes/blob/main/k8s-cluster-setup/metallb-l2-configuration.md">MetalLB Installation</a>)
- A NFS server as a storage class ( <a href="https://github.com/nasirnjs/kubernetes/blob/main/k8s-cluster-setup/dynamic-nfs-provisioning_k8s.md">Setup Dynamic NFS </a>)
- NFS client installed in all cluster nodes (<a href="https://bluexp.netapp.com/blog/azure-anf-blg-linux-nfs-server-how-to-set-up-server-and-client">NFS Client Setup</a>)
- Ingress controller in the cluster (<a href="https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx">Nginx Controller</a>)

  
## ☐ Setup Procedure : 
Create a Orgpod. This will create a namespace and an ingress-class-name. </br> 
Shift to the namespace that's been created by the Orgpod. 
```
kubens erpnext
```
Add frappe helm repo.
```
helm repo add frappe https://helm.erpnext.com
```
Before installing helm chart, edit the custom-values.yaml file.</br> 
Make sure :
✓ Proper image tag has been mentioned. </br> 
✓ Keep ```enabled : false``` in ingress, createsite, dropsite, backup and migrate. </br>    
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

### ☐ Note
✓ For upgrading new image ``` helm upgrade frappe-bench frappe/erpnext --reuse-values --set-string image.tag=0.0.15 --dry-run --debug ``` </br>
✓ Edit the custom.values.yaml having the right image tag and disable the jobs and others for creating template </br>
✓ Manually install custom ERP app ``` bench site aeslerp.com install-app custom_erp_app </br>

