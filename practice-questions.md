# Practice Question

## 1. Create a yaml for job that calculates the value of pi

Use the following as a command to run in the pod
```
command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
```

## 2. Create an Nginx Pod and attach an EmptyDir volume to it.

## 3. Create an Nginx deployment in the namespace “kube-cologne” and corresponding service of type NodePort . Service should be accessible on HTTP (80) and HTTPS (443)

## 4. Add label to a node as "arch=gpu"

## 5. Create a Role in the “conference” namespace to grant read access to pods.

## 6. Create a RoleBinding to grant the "pod-reader" role to a user "john" within the “conference” namespace.

## 7. Create an Horizontal Pod Autoscaler to automatically scale the Deployment if the CPU usage is above 50%.
    
Make sure you have heapster or metrics-server pods running in your cluster to gather the metrics.
    
Deploy a sample application

    kubectl run hpa-example--image=k8s.gcr.io/hpa-example --requests=cpu=200m --expose --port=80

 
 To test
 ```
    kubectl run -i --tty generate-load --image=busybox /bin/sh

    Inside container 

    while true; do wget -q -O- http://hpa-example.default.svc.cluster.local; done
```
Check HPA Status
    
```
kubectl get hpa
```

## 8. Deploy a default Network Policy for each resources in the default namespace to deny all ingress and egress traffic.
            
    To test try to ping one pod from another

## 9. Create a pod that contain multiple containers : nginx, redis, postgres with a single YAML file.

## 10. Deploy nginx application but with extra security using PodSecurityPolicy

    - Use runAsUser
    - Make the FileSystem read Only
    - Do not allow privilege escalation

To test exec into the pod 

```
kubectl exec -it security-context-demo -- sh

ps aux
```

You should see that PIDs will be 1000

## 11. Create a Config map from file.

Add the below lines in a file app.conf

```
      course=kubernetes
      subject=configmap
      title=ThisIsAConfigMapFromFile
```

## 12. Create a Pod using the busybox image to display the entire content of the above ConfigMap mounted as Volumes.

## 13. Create configmap from literal values

```
      course=kubernetes
      subject=configmap
      title=ThisIsAConfigMap
```

## 14. Create a Pod using the busybox image to display the above ConfigMap in environment variables automatically.

## 15. Create a ResourceQuota in a namespace "kube-cologn" that allows maximum of

 ```   
      - 10 ConfigMaps
      - 4 PVCs
      - 10 Secrets
      - 10 Services
      - 2 LoadBalancers
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

```
Maximum of 4 pods should be running in the namespace "pod-quota"

Only 4 replicas should be available
```

## 18. Deployment Exercise

a. Create nginx deployment and scale to 3

b. Check the history of the previous Nginx deployment

c. Update the Nginx version to the 1.9.1 in the previous deployment   

d. Check the history of the deployment to note the new entry

## 19. Add liveness and readiness probe to kuard container 

    For liveness probe, path used for health request is `/healthy` on port 8080
    For readiness probe, path used for ready request is `/ready` on port 8080
    Image: gcr.io/kuar-demo/kuard-amd64:1