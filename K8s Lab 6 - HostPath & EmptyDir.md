# Task 1 : Creating Deployment with hostPath
--------------------------------------------------------------------------
Steps to create pvdir:
SSH into the node:
```
ssh ubuntu@<node-ip>
```

Create the directory:
```
sudo mkdir -p /pvdir
```
```
cd /pvdir/
```
```
sudo chmod 777 /pvdir
```
```
touch file1 file2
```
```
exit
```
## Create a file named hp.yaml using content given below
```
vi hp.yaml
```
## Paste the following content into the file:
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: web-app
  name: web-app
spec:
  nodeName:  i-0f68a08663ec4e258
  volumes:
  - name: hp-volume
    hostPath:
      path: /pvdir
  containers:
  - image: nginx
    name: ctr-1
    ports:
    - containerPort: 80
    volumeMounts:
    - name: hp-volume
      mountPath: /usr/share/nginx/html
 ```
## Save and exit the editor
## Apply the Deployment yaml created in the previous step
```
kubectl apply -f hp.yaml
```
## View the objects created by Kubernetesand verify hostPath mounting in POD

```
kubectl get pods -o wide
```
```
kubectl describe pod <pod-name>
```
## Access shell on a container running in your Pod to verify volume
```
kubectl exec -it web-app -- /bin/bash
```
```
cd /usr/share/nginx/html/
```
```
ls
```
Cleanup
```
kubectl delete -f hp.yaml
```

# Task 2 : Creating Deployment with emptyDir
--------------------------------------------------------------------------
## Create a file named empty.yaml using content given below
```
vi empty.yaml
```
## Paste the following content into the file:
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: multi-ctr-app
  name: multi-ctr-app
spec:
  volumes:
  - name: emptydir-vol
    emptyDir: {}
  containers:
  - image: nginx
    name: ctr-1
    ports:
    - containerPort: 80
    volumeMounts:
    - name: emptydir-vol
      mountPath: /app
  - name: ctr-2
    image: busybox
    command: ["sh", "-c", "sleep 5000"]
    volumeMounts:
    - name: emptydir-vol
      mountPath: /data
```
## Save and exit the editor
## Apply the Deployment yaml created in the previous step
```
kubectl apply -f empty.yaml
```
## View the objects created by Kubernetes 
```
kubectl get pods
```
```
kubectl describe pod <pod-name>
```
## Access shell on a container running in your Pod to verify volume
```
kubectl exec -it <pod-name> -c <ctr-name> -- /bin/bash
```
```
kubectl exec -it <pod-name> -c <ctr-name> -- /bin/bash
```
## Check the emptydir
```
kubectl get pods -o wide
```
Identify the Pod's UID
```
kubectl get pod multi-ctr-app -o jsonpath='{.metadata.uid}'
```
Identify the node on which the pod is deployed and ssh into it
```
ssh ubuntu@nodepublicip
```
```
sudo su
```
```
cd /var/lib/kubelet/pods/
```
```
ls
```
## Enter the directory based on the noted POD's UID to find the emptydir volume

e.g.: /var/lib/kubelet/pods/856ef13f-26c7-4234-98d0-8df9c120beba/volumes/kubernetes.io~empty-dir/emptydir-vol
```
exit
```

# Task 3 Cleanup the resources using below command
-----------------------------------------------------------------------------
## Delete the resources created during the lab:
```
kubectl delete -f empty.yaml
```

