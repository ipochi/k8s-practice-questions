# Practice Question

## 1. Create a yaml for job that calculates the value of pi

Use the following as a command to run in the pod
```
command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
```

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  # activeDeadlineSeconds: 100
  # completions: 10
  # parallelism: 3
  template:
    metadata:
      name: pi
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never # Only Never or OnFailure is allowed
```
## 2. Create an Nginx Pod and attach an EmptyDir volume to it.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: nginx-storage
      mountPath: /data/test
  volumes:
  - name: nginx-storage
    emptyDir: {}
```

## 3. Create an Nginx deployment in the namespace “kube-cologne” and corresponding service of type NodePort . Service should be accessible on HTTP (80) and HTTPS (443)

```
apiVersion: v1
kind: Namespace
metadata:
  name: kube-cologne
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: kube-cologne
  labels:
    app: nginx
spec:
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
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: kube-cologne
spec:
  type: NodePort
  ports:
  - port: 80
    name: http
    targetPort: 80
  - port: 443
    name: https
    targetPort: 443
  selector:
    app: nginx
```

## 4. Add label to a node as "arch=gpu"
  
```
kubectl label nodes <node-name> arch=gpu
```

## 5. Create a Role in the “conference” namespace to grant read access to pods.

```
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n conference
```

## 6. Create a RoleBinding to grant the "pod-reader" role to a user "john" within the “conference” namespace.

```
kubectl create rolebinding read-pods --user=john --role=pod-reader --namespace conference
```

## 7. Create an Horizontal Pod Autoscaler to automatically scale the Deployment if the CPU usage is above 50%.
    
Make sure you have heapster or metrics-server pods running in your cluster to gather the metrics.
    
Deploy a sample application

    kubectl run hpa-example--image=k8s.gcr.io/hpa-example --requests=cpu=200m --expose --port=80

Create HPA based on CPU usage
    
    kubectl autoscale deployment hpa-example --cpu-percent=50 --min=1 --max=10

In another terminal  
    kubectl run -i --tty generate-load --image=busybox /bin/sh

Inside the above container run a loop bash command ito stress the CPU

    while true; do wget -q -O- http://hpa-example.default.svc.cluster.local; done

Check HPA Status
    
    kubectl get hpa

## 8. Deploy a default Network Policy for each resources in the default namespace to deny all ingress and egress traffic.
            
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-traffic
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```
    Try to ping one pod from another


## 9. Create a pod that contain multiple containers : nginx, redis, postgres with a single YAML file.
        
Create a multiple.yaml file with below contents

```
apiVersion: v1
kind: Pod
metadata:
  name: my-multi-app
  labels:
    env: formation
spec:
  containers:
  - name: nginx
    image: nginx
  - name: postgres
    image: postgres
  - name: redis
    image: redis
```
```
    kubectl create -f  multiple.yaml
```

## 10. Deploy nginx application but with extra security using PodSecurityPolicy

    - Use runAsUser
    - Make the FileSystem read Only
    - Do not allow privilege escalation
```
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: gcr.io/google-samples/node-hello:1.0
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
```
Exec into the pod 

```
kubectl exec -it security-context-demo -- sh

ps aux
```

You'll see that PIDs will be 1000

## 11. Create a Config map from file.

Add the below lines in a file app.conf

```
        course=kubernetes
        subject=configmap
        title=ThisIsAConfigMapFromFile
```
  
  
```
  kubectl create configmap cmap-from-file --from-file=app.conf
```

## 12. Create a Pod using the busybox image to display the entire content of the above ConfigMap mounted as Volumes.

```
apiVersion: v1
kind: Pod
metadata:
  name: cmap1
spec:
  containers:
    - name: busybox
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh","-c","cat /tmp/myconfigmap" ]
      volumeMounts:
      - name: my-config-map
        mountPath: /tmp/myconfigmap
  volumes:
    - name: my-config-map
      configMap:
        name: cmap-from-file
  restartPolicy: Never
```

## 13. Create configmap from literal values

```
        course=kubernetes
        subject=configmap
        title=ThisIsAConfigMap
```

```   
kubectl create configmap cmap --from-literal=course=kubernetes --from-literal=subject=configmap --from-literal=title=ThisIsAConfigMap
```

## 14. Create a Pod using the busybox image to display the entire ConfigMap in environment variables automatically.

```
apiVersion: v1
kind: Pod
metadata:
  name: cmap2
spec:
  containers:
    - name: busybox
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
          name: cmap
  restartPolicy: Never
```

## 15. Create a ResourceQuota in a namespace "kube-cologne" that allows maximum of

 ```   
        - 10 ConfigMaps
        - 4 PVCs
        - 10 Secrets
        - 10 Services
        - 2 LoadBalancers
```

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: max-count
  namespace: kube-cologne
spec:
  hard:
    configmaps: "10"
    persistentvolumeclaims: "4"
    secrets: "10"
    services: "10"
    services.loadbalancers: "2"   
```

Check with 

```
kubectl describe quota max-count --namespace=kube-cologne
```

## 16. Create ResourceQuota for a namespace "quota-namespace"

```
          - Total Memory Request for all containers must not exceed 1 GB
          - Total Memory Limit for all containers must not exceed 2 GB
          - CPU request total for all Containers must not exceed 1 cpu.
          - CPU limit total for all Containers must not exceed 2 cpu
```

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-namespace
  namespace: quota-namespace
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```
        

To Test , create pod1
```
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: quota-namespace
spec:
  containers:
  - name: pod1
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "800m" 
      requests:
        memory: "600Mi"
        cpu: "400m"
```

Attempt to create pod2
```
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  namespace: quota-namespace
spec:
  containers:
  - name: pod2
    image: redis
    resources:
      limits:
        memory: "1Gi"
        cpu: "800m"      
      requests:
        memory: "700Mi"
        cpu: "400m"
```

Pod2 creation should fail


## 17. Create Pod quota for a namespace "pod-quota"

Maximum of 4 pods should be running in the namespace "pod-quota"
```     
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-pod-namespace
  namespace: pod-quota
spec:
  hard:
    pods: "4"
```

Test with 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-quota-demo
  namespace: pod-quota
spec:
  selector:
    matchLabels:
      purpose: quota-demo
  replicas: 5
  template:
    metadata:
      labels:
        purpose: quota-demo
    spec:
      containers:
      - name: pod-quota-demo
        image: nginx
```

Only 4 replicaes will be available


## 18. Deployment Exercise

a. Create nginx deployment and scale to 3
```
    kubectl create deployment my-nginx --image=nginx
    kubectl scale deployment/my-nginx --replicas=3
```

b. Check the history of the previous Nginx deployment
```
    kubectl rollout history deployment my-nginx    
```    

c. Update the Nginx version to the 1.9.1 in the previous deployment
```
    kubectl set image deployment/my-nginx nginx=nginx:1.9.1
```    

d. Check the history of the deployment to note the new entry
```
        kubectl rollout history deployment my-nginx
```

## 19. Add liveness and readiness probe to kuard container 

    For liveness probe path used for health request is `/healthy` on port 8080
    For readiness probe path used for ready request is `/ready` on port 8080
    Image: gcr.io/kuar-demo/kuard-amd64:1

```
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      livenessProbe:
        httpGet:
          path: /healthy
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 30
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```