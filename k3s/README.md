# k3s

kubernetes related stuff

Installing k3s on RHEL 8

https://computingforgeeks.com/install-and-configure-ansible-awx-on-centos/

The steps from the link above and some of my own notes...

```bash

sudo dnf -y update
sudo reboot

# How to check if a reboot is actually needed

sudo dnf -y install yum-utils
sudo needs-restarting -r; echo $? # Check to see if reboot is actually required. If needs-restarting returns 1 reboot is needed.

sudo setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
cat /etc/selinux/config | grep SELINUX=

curl -sfL https://get.k3s.io | sudo bash -
systemctl status k3s.service

sudo su -

kubectl get nodes
kubectl version --short
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/devel/deploy/awx-operator.yaml
kubectl get pods
kubectl create ns awx
vi public-static-pvc.yaml

```


```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: public-static-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 5Gi
```

kubectl apply -f public-static-pvc.yaml -n awx

```bash

kubectl get pvc -n awx

vi awx-instance-deployment.yml
```

```yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  service_type: nodeport
  projects_persistence: true
  projects_storage_access_mode: ReadWriteOnce
  web_extra_volume_mounts: |
    - name: static-data
      mountPath: /var/lib/awx/public
  extra_volumes: |
    - name: static-data
      persistentVolumeClaim:
        claimName: public-static-data-pvc
```

```bash
kubectl apply -f awx-instance-deployment.yml -n awx
kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator" -n awx
kubectl  get pvc

# If there is a problem (most likely there is) with access for postgresql, do the following
kubectl logs awx-postgres-0
#mkdir: cannot create directory ‘/var/lib/postgresql/data’: Permission denied

ls -lh /var/lib/rancher/k3s/storage/ | grep awx-postgres-0

chmod -R 777  /var/lib/rancher/k3s/storage/*
kubectl delete pods -l "app.kubernetes.io/managed-by=awx-operator" -n awx

kubectl get pods -n awx

kubectl get service -n awx

http://your-server-ip-address:nnnnn

# For the password

kubectl -n awx get secret awx-admin-password -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'

```
