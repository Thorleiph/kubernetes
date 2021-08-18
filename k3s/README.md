# k3s

kubernetes related stuff

Installing k3s on RHEL 8

https://computingforgeeks.com/install-and-configure-ansible-awx-on-centos/

The steps from the link above and some of my own notes...

sudo dnf -y update

sudo reboot



sudo dnf -y install yum-utils

sudo needs-restarting -r; echo $?

Check to see if reboot is actually required. If needs-restarting returns 1 reboot is needed.


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

'''yaml
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
'''
