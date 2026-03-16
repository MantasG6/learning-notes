COURSE:        Storage for CKA
AUTHOR:        Antonio Jesus Piedra
______________________________________

# Demo: Dynamic Volume Provisioning - WaitForFirstConsumer

This guide demonstrates how to use a StorageClass with WaitForFirstConsumer binding mode, 
which delays volume provisioning until a Pod using the PVC is created.


## Steps

1. Switch to module3 context:
   ```bash
   kubectl config use-context module3-context
   ```

2. Review the [StorageClass definition](delayed-binding-sc.yaml):
   - Uses minikube hostpath provisioner
   - Sets volumeBindingMode to WaitForFirstConsumer
   - Named 'delayed-binding'
   - Not specifying reclaimPolicy, defaults to Delete

3. Create the StorageClass:
   ```bash
   kubectl apply -f delayed-binding-sc.yaml
   ```

4. Review the [PVC definition](pvc-delayed.yaml):
   - Requests 1Gi of storage
   - ReadWriteOnce access mode
   - Uses the 'delayed-binding' StorageClass

5. Create the PVC:
   ```bash
   kubectl apply -f pvc-delayed.yaml
   ```

6. Check the PVC status:
   ```bash
   kubectl get pvc
   ```
   Note: The PVC remains in Pending state because no Pod is using it yet

7. Ensure no PV was created
   ```bash
   kubectl get pv
   ```

7. Review the [Pod definition](pod.yaml):
   - Uses the PVC we just created
   - Specifies a node selector for demonstration purposes

8. Create the Pod:
   ```bash
   kubectl apply -f pod.yaml
   ```

9. Watch the PVC and Pod status:
   ```bash
   kubectl get pod,pvc
   ```
   You'll see:
   - PVC: STATUS changes from Pending to Bound
   - Pod: STATUS changes from Pending to Running

10. Check the automatically created PV:
   ```bash
   kubectl get pv
   ```
   Note:
   - PV was created only after Pod scheduling (avoids unused volumes & efficient resource usage)
   - The volume is created on the node where the Pod is scheduled (better scheduling decissions)


## Cleanup

1. Delete the Pod:
   ```bash
   kubectl delete pod delayed-pod
   ```

2. Delete the PVC:
   ```bash
   kubectl delete pvc my-delayed-pvc
   ```

3. Delete the StorageClass:
   ```bash
   kubectl delete sc delayed-binding-sc
   ```