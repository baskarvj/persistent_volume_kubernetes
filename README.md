## Create IAM role for service Account with EBS policy

create role -->  web_identity --> select your cluster oidc --> select policy --> AmazonEBSCSIDriverRole --> give anme for role -->  AmazonEBSCSIDriverRole



## create value file for EBS CSI servicve account 
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
      eks.amazonaws.com/role-arn: <Enter your ARN of the EBS CSI IAM Role that created the above step>
~~~


## Add EBS CSI driver chart to helm repo
~~~
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
~~~

## Install EBS CSI Driver  
~~~
helm install aws-ebs-csi-driver aws-ebs-csi-driver/aws-ebs-csi-driver -n kube-system --values=ebs-csi.yml
~~~

## Create storage class
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


## Create PVC 
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

## Create pod with Persistent Volume
~~~
vi pod.yml
~~~
~~~
kind: Pod
apiVersion: v1
metadata:
  name: custompod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: external
  volumes:
    - name: external
      persistentVolumeClaim:
        claimName: block-pvc
~~~
