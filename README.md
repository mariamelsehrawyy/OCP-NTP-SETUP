# OCP-NTP-SETUP
## This repository documents how to apply/overwrite your NTP servers on openshift cluster by using machineconfig.


### Walking up to find the below alarm screaming, remembering that your production cluster is configured with single NTP node and it had unexpected shutdown or you didn't configure the NTP servers properly while implementing your cluster.
<img width="1440" height="635" alt="Screen Shot 2026-01-04 at 1 04 39 AM" src="https://github.com/user-attachments/assets/c6a13a9c-deff-4716-96bf-9337ecc9df1d" />

### Step 1 - On the bastion machine create chrony.conf file and put your configuartion as below
a. vim chrony.conf
```
server <INTERNAL_NTP_IP> iburst prefer
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync

```

b. replace <INTERNAL_NTP_IP> by your NTP IPs

### Step 2 - Encode the chrony.conf file by using the below command and copy the output.

```
base64 -w0 chrony.conf

```

### Step 3 -  Create your MachineConfig for WORKERS and paste the above encoded output on <BASE64_OUTPUT> 

```
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: 99-worker-chrony
  labels:
    machineconfiguration.openshift.io/role: worker
spec:
  config:
    ignition:
      version: 3.2.0
    storage:
      files:
      - path: /etc/chrony.conf
        mode: 0644
        overwrite: true
        contents:
          source: data:text/plain;base64,<BASE64_OUTPUT>
```
### Step 4 - Apply machine config on workers
```
oc apply -f mc-worker-chrony.yaml
```

### Step 5 - Watch rollout on all workers, make sure that all workers will reboot.
```
oc get mcp worker -w
oc get mcp -w  //for both master and worker pool

```

### Step 6 Apply the above again for master machines - to test and validate that configuartion has been applied successfully, login to the machines, the output should appear your NTP configurations/IP


```
cat /etc/chrony.conf
chronyc sources

```

### make sure that all your alarms has been cleared after the action :)!


<img width="1440" height="632" alt="Screen Shot 2026-01-04 at 4 05 18 AM" src="https://github.com/user-attachments/assets/71785e15-8b44-4bca-bfbd-aba6cc768499" />










