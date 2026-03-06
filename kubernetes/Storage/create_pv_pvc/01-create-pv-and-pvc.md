COURSE:        Storage for CKA
AUTHOR:        Antonio Jesus Piedra
______________________________________

# Creating Persistent Volumes and Persistent Volume Claims

## Prerequisites

Ensure you have completed the cluster setup instructions in the [setup guide](README.md).

## Steps

1. Switch to module1 context:
   ```bash
   kubectl config use-context module1-context
   ```

2. Review the [PV definition](pv.yaml):
   - A 2Gi storage capacity
   - ReadWriteOnce access mode
   - hostPath storage at "/data/my-pv"

3. Create the Persistent Volume (PV):
   ```bash
   kubectl apply -f pv.yaml
   ```

4. Verify the PV creation:
   ```bash
   kubectl get pv
   ```

5. Review the [PVC definition](pvc.yaml):
   - Requests 2Gi of storage and
   - ReadWriteOnce access mode

6. Create the Persistent Volume Claim (PVC):
   ```bash
   kubectl apply -f pvc.yaml
   ```

7. Verify the PVC creation and binding:`
   ```bash
   kubectl get pvc
   ```

8. Ensure the PVC is bound to the correct PV:
   ```bash
   kubectl get pv
   ```
