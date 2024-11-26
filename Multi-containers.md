## Multi-container

### Task 1: Sidecar container
```
vi sidecar.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - containerPort: 80
    # Main application container

  - name: sidecar-container
    image: debian:latest
    command: ['sh', '-c', 'apt update && apt install curl -y && while true; do echo "Sidecar Running"; sleep 10; done']
    # Sidecar container
```
```	
kubectl apply -f sidecar.yaml
```
```
kubectl get pod
```
```
kubectl exec -it sidecar-pod -c sidecar-container -- sh
```
``` 
kubectl logs sidecar-pod -c sidecar-container
```


### Task 2: Init container
```
vi init.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /app
    # Main application container

  initContainers:
  - name: init-container
    image: busybox:latest
    command: ['sh', '-c', 'echo "Init Container Completed" > /work-dir/completed.txt']
    volumeMounts:
    - name: workdir
      mountPath: /work-dir
    # Init container
  volumes:
  - name: workdir
    emptyDir: {}

```
```	
kubectl apply -f init.yaml
```
```
kubectl get pod
```
![image](https://github.com/user-attachments/assets/4aa61bfd-9bfd-43da-8e87-f6e78fcdb3ed)


Go and update the command for the init container to write the file after a lapse of 60 seconds. You would notice the main container starting only after init container has completed its task
```
command: ['sh', '-c', 'sleep 60 && echo "Init Container Completed" > /work-dir/completed.txt']
```
Replace the pod
```
kubectl replace -f init-pod.yaml --force
```
```
kubectl get po
```
Notice the PodInitializing status coming up only after a lapse of 60 seconds 
![image](https://github.com/user-attachments/assets/445d7c73-9766-46a7-be43-c7f8e9ad2ebb)

```
kubectl exec -it init-pod -c main-container -- bash
```
```
cd /app && ls
```
```
cat completed.txt
```
```
exit
```

