## Step 1: Verify the Default StorageClass
Check if a StorageClass exists and supports dynamic provisioning:
```
kubectl get storageclass
```
Expected Output:
![image](https://github.com/user-attachments/assets/b01b065f-4121-4037-ae49-c856fe05fb82)

```
vi pvc-dynamic.yaml
```
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```
## Step 2: Verify PVC and PV Binding
Since the kops-csi-1-21 StorageClass uses WaitForFirstConsumer, the PersistentVolume (PV) will not be provisioned until a pod is scheduled to use it. Check the PVC status:
```
kubectl get pvc
```
Expected output:

NAME          STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS      AGE
dynamic-pvc   Pending   <none>   <none>     <none>         kops-csi-1-21    10s

This indicates that the PV is waiting for a pod to bind to it.

## Step 3: Create a Deployment Using the PVC
Create a deployment that uses the PVC. Save the following YAML as nginx-deployment.yaml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: pvc-storage
      volumes:
      - name: pvc-storage
        persistentVolumeClaim:
          claimName: dynamic-pvc
```
Apply the deployment:
```
kubectl apply -f nginx-deployment.yaml
```

## Step 4: Verify Resource Binding
Check the PVC status again:
```
kubectl get pvc
```
The STATUS should change to Bound.

Check the dynamically provisioned PV:
```
kubectl get pv
```
Verify the pod is running:
```
kubectl get pods
```

## Step 5: Test the Storage
Access the pod:

```
kubectl exec -it <pod-name> -- /bin/bash
```
Navigate to the mounted volume:
```
cd /usr/share/nginx/html
```
Create a test file:
```
echo "Kops CSI Dynamic Provisioning Test" > index.html
```
Exit the pod:
```
exit
```
Port-forward to access the pod:
```
kubectl port-forward <pod-name> 8080:80
```

## Open a browser and navigate to http://localhost:8080. You should see the content of the index.html file.

## Step 6: Clean Up
To remove all created resources:
```
kubectl delete deploy nginx-deployment
kubectl delete pvc dynamic-pvc
```
