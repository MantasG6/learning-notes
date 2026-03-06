COURSE:        Storage for CKA
AUTHOR:        Antonio Jesus Piedra
______________________________________

# Deploying a Stateful Application with Persistent Storage

This guide demonstrates how persistent storage works by deploying a 
simple NGINX web server that uses our previously created PVC.


## Steps

1. Switch to module1 context:
   ```bash
   kubectl config use-context module1-context
   ```

2. Review the [Pod definition](pod.yaml):
    - Volume definition that references our PVC
    - Volume mount that connects the storage to NGINX's web root

3. Deploy the NGINX Pod:
   ```bash
   kubectl apply -f pod.yaml
   ```

4. Wait for the Pod to be ready:
   ```bash
   kubectl get pod storage-test-pod -w
   ```

5. Create a test file in the persistent volume:
   ```bash
   kubectl exec -it storage-test-pod -- /bin/bash -c \
     "echo 'Hello from persistent storage!' > /usr/share/nginx/html/index.html"
   ```

## Testing Persistence

1. Verify the file was created:
   ```bash
   kubectl exec -it storage-test-pod -- cat /usr/share/nginx/html/index.html
   ```

2. Simulate a failure by deleting the Pod:
   ```bash
   kubectl delete pod storage-test-pod
   ```

3. Verify storage resources are intact:
   ```bash
   kubectl get pv,pvc
   ```
   Note: You should see that both PV and PVC are still bound, unaffected by the Pod deletion.

4. Recreate the Pod:
   ```bash
   kubectl apply -f pod.yaml
   ```

5. Verify the data persisted:
   ```bash
   kubectl exec -it storage-test-pod -- cat /usr/share/nginx/html/index.html
   ```
   Expected output: "Hello from persistent storage!"

## Cleanup

Pod:
```bash
kubectl delete pod storage-test-pod
```

Persistent Volume Claim:
```bash
kubectl delete pvc my-pvc
```

Persistent Volume:
```bash
kubectl delete pv my-pv
```
