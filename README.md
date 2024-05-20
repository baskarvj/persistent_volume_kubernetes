## For EBS (persistent Volume) CSI Driver 
________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________

### <span style="color:skyblue">Create IAM role for service Account with EBS policy</span>

create role -->  web_identity --> select your cluster oidc --> select policy --> AmazonEBSCSIDriverPolicy --> give name for role -->  AmazonEBSCSIDriverRole

`

### create value file for EBS CSI servicve account 
`
~~~
vi ebs.yml
~~~
~~~
controller:
  serviceAccount:
    create: true
    name: ebs-csi-controller
    nameTest:
    annotations:
      eks.amazonaws.com/role-arn: <Enter_your_EBS_CSI_IAM_Role_ARN_here>
~~~

### Add EBS CSI driver chart to helm repo
~~~
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
~~~

### Install EBS CSI Driver  
~~~
helm install aws-ebs-csi-driver aws-ebs-csi-driver/aws-ebs-csi-driver -n kube-system --values=ebs-csi.yml
~~~

### Create storage class
~~~
vi sc.yml
~~~
~~~
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: kubernetes.io/aws-ebs
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: false
parameters:
  type: gp2
  fsType: xfs
~~~
~~~
kubectl apply -f sc.yml
~~~

### Create PVC 
~~~
vi pvc.yml
~~~
~~~
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-sc
  volumeMode: Block
  resources:
    requests:
      storage: 5Gi
~~~
~~~
kubectl apply -f pvc.yml
~~~
### Create pod with Persistent Volume
~~~
vi pod.yml
~~~
~~~
kind: Pod
apiVersion: v1
metadata:
  name: pv-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: my-vol
  volumes:
    - name: my-vol
      persistentVolumeClaim:
        claimName: block-pvc
~~~
~~~
kubectl apply -f pod.yml
~~~

________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________


### <span style="color:skyblue">For EFS (persistent Volume) CSI Driver</span>

### Create IAM role for service Account with EfS policy

create role -->  web_identity --> select your cluster oidc --> select policy --> AmazonEFSCSIDriverPolicy --> give name for role -->  AmazonEFSCSIDriverRole



### create value file for EFS CSI servicve account 
~~~
vi ebs.yml
~~~
~~~
controller:
  serviceAccount:
    create: true
    name: efs-csi-controller
    nameTest:
    annotations:
      eks.amazonaws.com/role-arn: <Enter_your_EFS_CSI_IAM_Role_ARN_here>
~~~

### Add EFS CSI driver chart to helm repo
~~~
helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver
~~~

### Install EBS CSI Driver  
~~~
helm install aws-efs-csi-driver aws-efs-csi-driver/aws-efs-csi-driver -n kube-system --values=efs.yml

~~~

### Create storage class
~~~
vi sc.yml
~~~
~~~
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: kubernetes.io/aws-ebs
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: false
parameters:
  type: gp2
  fsType: xfs
~~~
~~~
kubectl apply -f sc.yml
~~~

### Create PVC 
~~~
vi pvc.yml
~~~
~~~
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-sc
  volumeMode: Block
  resources:
    requests:
      storage: 5Gi
~~~
~~~
kubectl apply -f pvc.yml
~~~
### Create pod with Persistent Volume
~~~
vi pod.yml
~~~
~~~
kind: Pod
apiVersion: v1
metadata:
  name: pv-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: my-vol
  volumes:
    - name: my-vol
      persistentVolumeClaim:
        claimName: block-pvc
~~~
~~~
kubectl apply -f pod.yml
~~~

