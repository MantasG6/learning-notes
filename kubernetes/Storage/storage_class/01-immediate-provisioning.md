COURSE:        Storage for CKA
AUTHOR:        Antonio Jesus Piedra
______________________________________

# Demo: Dynamic Volume Provisioning - Immediate Binding

This guide demonstrates how the default StorageClass with immediate binding mode provisions PersistentVolumes as soon as a PVC is created.

## Steps

1. Switch to module3 context:
   ```bash
   kubectl config use-context module3-context
   ```

2. View available StorageClasses:
   ```bash
   kubectl get storageclass
   ```
   You should see the default StorageClass marked with `(default)`

3. Examine the default StorageClass:
   ```bash
   kubectl describe sc standard
   ```
   Note the following details:
   - The provisioner being used
   - ReclaimPolicy (typically set to Delete)
   - VolumeBindingMode: Immediate (volume is provisioned immediately when PVC is created)

4. Review the [PVC definition](pvc-inmediate.yaml):
   - Requests 1Gi of storage
   - ReadWriteOnce access mode
   - No storageClassName specified (will use default)

5. Create the PVC:
   ```bash
   kubectl apply -f pvc-inmediate.yaml
   ```

6. Check the PVC status:
   ```bash
   kubectl get pvc
   ```
   Note: STATUS should be Bound after a few seconds

7. Check the automatically created PV:
   ```bash
   kubectl get pv
   ```
   Note:
   - A new PV has been created automatically
   - It's bound to your PVC
   - The size matches your request (1Gi)
   - No need to create a Pod first
   - Storage is reserved even if not used yet


## Cleanup

Delete the PVC:
```bash
kubectl delete pvc my-inmediate-pvc
```
Note: The PV will be automatically deleted due to the Delete reclaim policy.

Ensure the PV has been deleted automatically:
```bash
kubectl get pv 
```
