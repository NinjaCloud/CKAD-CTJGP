## Resource Quotas
In Kubernetes, a ResourceQuota is a way to limit the amount of resources (CPU, memory, and persistent storage) that a namespace can consume. It helps in preventing resource exhaustion and ensures that one namespace doesn't negatively impact the performance or stability of others.

Keep in mind that ResourceQuotas are a way to define constraints, and they do not actively enforce them on existing resources. They are checked when new resources are created or when existing resources are updated. If a namespace exceeds its quota, the creation or update of resources may be rejected.

### Task 1: Creating a Namespace

```
kubectl create namespace ns1
```
```
kubectl get ns
```



### Task 2: Creating Resource Quota and Constraining Object Creation

Create a pod and expose it, before applying the resource quota to check if the resource quota applies to existing objects or not.
```
kubectl -n ns1 run pod1 --image nginx --port 80
```
```
kubectl -n ns1 expose pod pod1 --name pod1-svc --port 80 --type NodePort
```
```
kubectl -n ns1 run pod2 --image nginx --port 80
```
```
kubectl -n ns1 expose pod pod2 --name pod2-svc --port 80 --type NodePort
```



```
kubectl get quota -n ns1
```
![image](https://github.com/user-attachments/assets/21503318-e6b6-4b13-ad48-a0663f081b9a)

Declarative : We will create another resource quota in the same namespace through this method.
```
vi rq2.yaml
```
```yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: rs-quota2
  namespace: ns1
spec:
  hard:
    pods: "2"
    services: "2"

```
```
kubectl apply -f rq2.yaml
```
```
kubectl describe ns ns1
```
![image](https://github.com/user-attachments/assets/32287061-4b45-4457-b1cd-879ad57078e5)

```
kubectl get quota -n ns1
```
![image](https://github.com/user-attachments/assets/ed11fb6f-f853-40cd-8b53-722dce8ea637)

Try deploying a new pod in namespace ns1
```
kubectl -n ns1 run pod3 --image nginx --port 80
```
![image](https://github.com/user-attachments/assets/fe9ea4d5-2948-4f22-bf6c-4db894316e93)

Try creating a new service by exposing the existing pod.
```
kubectl -n ns1 expose pod pod2 --name pod21-svc --port 80 --type NodePort
```
![image](https://github.com/user-attachments/assets/1a8cf986-bb1f-4708-aa50-268b2710d4d9)


### Task 3: Creating Resource Quota and Constraining Hardware Resources

```
vi rq3.yaml
```
```yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: rs-quota3
  namespace: ns1
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```
```
kubectl apply -f rq3.yaml
```
```
kubectl describe ns ns1
```
Try and create a pod in the namespace. 
```
kubectl -n ns1 run pod4 --image nginx --port 80
```
![image](https://github.com/user-attachments/assets/a0f2eb55-7a3b-43a5-a5ad-ec1b82adb994)

```
vi rq4.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod5
  namespace: ns1
spec:
  containers:
  - name: quota-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "1000m"
      requests:
        memory: "600Mi"
        cpu: "350m"
    ports:
      - containerPort: 80
```
```	  
kubectl apply -f rq4.yaml
```
```
kubectl describe ns ns1
```
```
kubectl get resourcequota -n ns1 -o yaml
```
Deploy another Pod, such that the total resource exceeds the resoucre quota limit
```
vi rq5.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod6
  namespace: ns1
spec:
  containers:
  - name: quota-ctr
    image: nginx
    resources:
      limits:
        memory: "1600Mi"
        cpu: "2000m"
      requests:
        memory: "1600Mi"
        cpu: "800m"
    ports:
      - containerPort: 80
```
```
kubectl apply -f rq5.yaml
```
![image](https://github.com/user-attachments/assets/b0a0415d-cd3c-4a37-a4b9-916dbc994dcf)
