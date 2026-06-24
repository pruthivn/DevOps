# Kubernetes Stateful set:
In the below picture you can see the 6/6 in Ready column for pods "ebs-csi-controller" means 6 containers are running that pod if we use describe pod command for "ebs-csi-controller" in kube-system namspace you will get detailed info of those 6 containers.
A pod can contain N number of containers.
![alt text](.images/image.png)

## manifest