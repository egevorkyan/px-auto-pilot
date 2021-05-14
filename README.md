# PortWorx Autopilot feature for volume autoexpansion in Kubernetes
OpenShift is my famous Kubernetes Platform, that's why all demos are done on OpenShift Cluster but
same things can work on different Kubernetes as well.

Portworx is advanced and amazing storage platform which supports all major Kubernetes Platforms.

- Requiremts for this demo:
1. Kubernetes version 1.11 or higher. Currentrly used: OCP Version: 4.7.9 Kubernetes Version: v1.20.0+7d0a2b2
2. Installed PortWorx storage cluster with autopilot feature enabled (Portworx Operator was used in this demo to deploy portworx storage)
3. Prometheus installed and configured proper, also integreted with autopilot

Checking prerequisites:
```bash
redhat2020: oc get nodes
NAME                        STATUS   ROLES          AGE   VERSION
demo-hvwtp-master-0         Ready    master         21d   v1.20.0+7d0a2b2
demo-hvwtp-master-1         Ready    master         21d   v1.20.0+7d0a2b2
demo-hvwtp-master-2         Ready    master         21d   v1.20.0+7d0a2b2
demo-hvwtp-portworx-7446w   Ready    storage        40h   v1.20.0+7d0a2b2
demo-hvwtp-portworx-ngq2c   Ready    storage        40h   v1.20.0+7d0a2b2
demo-hvwtp-portworx-t787c   Ready    storage        40h   v1.20.0+7d0a2b2
demo-hvwtp-worker-0-9zv2g   Ready    infra,worker   21d   v1.20.0+7d0a2b2
demo-hvwtp-worker-0-s48ck   Ready    worker         11d   v1.20.0+7d0a2b2
demo-hvwtp-worker-0-zmjsl   Ready    infra,worker   21d   v1.20.0+7d0a2b2

redhat2020: oc get pods -n portworx
NAME                                                 READY   STATUS    RESTARTS   AGE
autopilot-ff7ff4694-8vg44                            1/1     Running   0          35h
portworx-api-jmfnz                                   1/1     Running   0          35h
portworx-api-pfvnl                                   1/1     Running   0          35h
portworx-api-s6nnt                                   1/1     Running   0          35h
portworx-pvc-controller-5c4d7f7944-5s6wh             1/1     Running   0          35h
portworx-pvc-controller-5c4d7f7944-9l6t9             1/1     Running   0          35h
portworx-pvc-controller-5c4d7f7944-k8xwz             1/1     Running   0          35h
prometheus-px-prometheus-0                           3/3     Running   0          35h
px-csi-ext-5487f5cd5d-9twkf                          3/3     Running   0          24h
px-csi-ext-5487f5cd5d-fg7jp                          3/3     Running   0          24h
px-csi-ext-5487f5cd5d-n95hh                          3/3     Running   0          24h
px-demo-8d98be61-38bf-4ff5-b8cb-5fddf2f49eda-4g424   2/2     Running   0          24h
px-demo-8d98be61-38bf-4ff5-b8cb-5fddf2f49eda-jqx65   2/2     Running   0          24h
px-demo-8d98be61-38bf-4ff5-b8cb-5fddf2f49eda-lfmcd   2/2     Running   0          24h
px-lighthouse-c758bdcc8-n5d9x                        3/3     Running   0          35h
px-prometheus-operator-5c668f65c5-p4hcs              1/1     Running   0          35h
stork-5c666bcd5-brf4t                                1/1     Running   0          24h
stork-5c666bcd5-cmtnb                                1/1     Running   0          24h
stork-5c666bcd5-f4wlc                                1/1     Running   0          24h
stork-scheduler-d6b688584-8ggst                      1/1     Running   0          35h
stork-scheduler-d6b688584-8lm5s                      1/1     Running   0          35h
stork-scheduler-d6b688584-nwzzm                      1/1     Running   0          35h

redhat2020: oc get cm -n portworx
NAME                                   DATA   AGE
autopilot-config                       1      35h
kube-root-ca.crt                       1      41h
prometheus-px-prometheus-rulefiles-0   1      35h

redhat2020: oc exec -n portworx px-demo-8d98be61-38bf-4ff5-b8cb-5fddf2f49eda-4g424 -- /opt/pwx/bin/pxctl status

Status: PX is operational
License: Trial (expires in 29 days)
Node ID: 7f3cff9c-45d8-4506-9e37-735ca8855a61
        IP: 10.10.0.19
        Local Storage Pool: 2 pools
        POOL    IO_PRIORITY     RAID_LEVEL      USABLE  USED    STATUS  ZONE    REGION
        0       HIGH            raid0           15 GiB  3.0 GiB Online  nova    regionOne
        1       HIGH            raid0           37 GiB  3.9 GiB Online  nova    regionOne
        Local Storage Devices: 3 devices
        Device  Path            Media Type              Size            Last-Scan
        0:1     /dev/vdc        STORAGE_MEDIUM_MAGNETIC 15 GiB          13 May 21 07:29 UTC
        1:1     /dev/vdd2       STORAGE_MEDIUM_MAGNETIC 17 GiB          13 May 21 07:29 UTC
        1:2     /dev/vdb        STORAGE_MEDIUM_MAGNETIC 20 GiB          13 May 21 07:29 UTC
        total                   -                       52 GiB
        Cache Devices:
         * No cache devices
        Journal Device:
        1       /dev/vdd1       STORAGE_MEDIUM_MAGNETIC
Cluster Summary
        Cluster ID: px-demo-8d98be61-38bf-4ff5-b8cb-5fddf2f49eda
        Cluster UUID: 0751de85-c3bf-4e2c-a6f7-cf3df91895be
        Scheduler: kubernetes
        Nodes: 3 node(s) with storage (3 online)
        IP              ID                                      SchedulerNodeName               Auth            StorageNode     Used    Capacity        Status  StorageStatus   Version         Kernel                              OS
        10.10.0.19      7f3cff9c-45d8-4506-9e37-735ca8855a61    demo-hvwtp-portworx-t787c       Disabled        Yes             6.9 GiB 52 GiB          Online  Up (This node)  2.7.1.0-8a9e965 4.18.0-240.22.1.el8_3.x86_64        Red Hat Enterprise Linux CoreOS 47.83.202104250838-0 (Ootpa)
        10.10.0.44      419c47c4-7ad3-4263-9901-b0083d01a852    demo-hvwtp-portworx-ngq2c       Disabled        Yes             6.9 GiB 52 GiB          Online  Up              2.7.1.0-8a9e965 4.18.0-240.22.1.el8_3.x86_64        Red Hat Enterprise Linux CoreOS 47.83.202104250838-0 (Ootpa)
        10.10.0.11      0d558f48-5800-475b-9aa3-2a0417620fd5    demo-hvwtp-portworx-7446w       Disabled        Yes             6.9 GiB 52 GiB          Online  Up              2.7.1.0-8a9e965 4.18.0-240.22.1.el8_3.x86_64        Red Hat Enterprise Linux CoreOS 47.83.202104250838-0 (Ootpa)
Global Storage Pool
        Total Used      :  21 GiB
        Total Capacity  :  156 GiB
```

Based on the checking prerequisites, infrastructure is ready to demonstrate autopilot feature

# Deploying PGBench application to check how PortWorx Autopilot will work
- Use manifests from manifests folder and execute it - Results:
```bash
redhat2020: oc get projects demo-pgsql-01
NAME            DISPLAY NAME   STATUS
demo-pgsql-01                  Active
redhat2020: oc get projects demo-pgsql-02
NAME            DISPLAY NAME   STATUS
demo-pgsql-02                  Active

redhat2020: oc get sc
NAME                             PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
demo-autopilot-pgsql-sc          kubernetes.io/portworx-volume   Delete          Immediate              true                   47m

redhat2020: oc get pvc --all-namespaces
NAMESPACE       NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              AGE
demo-pgsql-01   pgbench-data    Bound    pvc-b253ea76-8032-4f36-b5ea-fdc2085534f1   10Gi       RWO            demo-autopilot-pgsql-sc   35s
demo-pgsql-01   pgbench-state   Bound    pvc-85aab609-e37d-4035-a529-71d4a1533dbf   1Gi        RWO            demo-autopilot-pgsql-sc   35s
demo-pgsql-02   pgbench-data    Bound    pvc-aa931830-f795-43d2-9fa0-9bd74338a043   10Gi       RWO            demo-autopilot-pgsql-sc   42s
demo-pgsql-02   pgbench-state   Bound    pvc-0fca125a-3ff4-473b-9176-e68e7f6161e4   1Gi        RWO            demo-autopilot-pgsql-sc   41s

redhat2020: oc exec -n portworx px-demo-8d98be61-38bf-4ff5-b8cb-5fddf2f49eda-4g424 -- /opt/pwx/bin/pxctl volume list
ID                      NAME                                            SIZE    HA      SHARED  ENCRYPTED       PROXY-VOLUME    IO_PRIORITY     STATUS          SNAP-ENABLED
188908979085679416      pvc-0fca125a-3ff4-473b-9176-e68e7f6161e4        1 GiB   2       no      no              no              HIGH            up - detached   no
546428899120468870      pvc-85aab609-e37d-4035-a529-71d4a1533dbf        1 GiB   2       no      no              no              HIGH            up - detached   no
531474702257140723      pvc-aa931830-f795-43d2-9fa0-9bd74338a043        10 GiB  2       no      no              no              HIGH            up - detached   no
995533517199009688      pvc-b253ea76-8032-4f36-b5ea-fdc2085534f1        10 GiB  2       no      no              no              HIGH            up - detached   no

redhat2020: oc get autopilotrule
NAME                 AGE
demo-volume-resize   16s
```
- Portworx Autopilot work results
```bash
redhat2020: oc get pods -l app=pgbench --all-namespaces
NAMESPACE       NAME                       READY   STATUS    RESTARTS   AGE
demo-pgsql-01   pgbench-7b94cb5cf4-tzzzv   2/2     Running   1          4m19s
demo-pgsql-02   pgbench-7b94cb5cf4-krfks   2/2     Running   1          4m24s

redhat2020: oc get pvc --all-namespaces
NAMESPACE       NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS              AGE
demo-pgsql-01   pgbench-data    Bound    pvc-b253ea76-8032-4f36-b5ea-fdc2085534f1   20Gi       RWO            demo-autopilot-pgsql-sc   13m
demo-pgsql-01   pgbench-state   Bound    pvc-85aab609-e37d-4035-a529-71d4a1533dbf   1Gi        RWO            demo-autopilot-pgsql-sc   13m
demo-pgsql-02   pgbench-data    Bound    pvc-aa931830-f795-43d2-9fa0-9bd74338a043   20Gi       RWO            demo-autopilot-pgsql-sc   13m
demo-pgsql-02   pgbench-state   Bound    pvc-0fca125a-3ff4-473b-9176-e68e7f6161e4   1Gi        RWO            demo-autopilot-pgsql-sc   13m

redhat2020: oc exec -n portworx px-demo-8d98be61-38bf-4ff5-b8cb-5fddf2f49eda-4g424 -- /opt/pwx/bin/pxctl volume list
ID                      NAME                                            SIZE    HA      SHARED  ENCRYPTED       PROXY-VOLUME    IO_PRIORITY     STATUS                          SNAP-ENABLED
188908979085679416      pvc-0fca125a-3ff4-473b-9176-e68e7f6161e4        1 GiB   2       no      no              no              HIGH            up - attached on 10.10.0.11     no
546428899120468870      pvc-85aab609-e37d-4035-a529-71d4a1533dbf        1 GiB   2       no      no              no              HIGH            up - attached on 10.10.0.19     no
531474702257140723      pvc-aa931830-f795-43d2-9fa0-9bd74338a043        20 GiB  2       no      no              no              HIGH            up - attached on 10.10.0.11     no
995533517199009688      pvc-b253ea76-8032-4f36-b5ea-fdc2085534f1        20 GiB  2       no      no              no              HIGH            up - attached on 10.10.0.19     no

redhat2020: oc get events --field-selector involvedObject.kind=AutopilotRule,involvedObject.name=demo-volume-resize --all-namespaces --sort-by .lastTimestamp
default     14m         Normal   Transition   autopilotrule/demo-volume-resize   rule: demo-volume-resize:pvc-aa931830-f795-43d2-9fa0-9bd74338a043 transition from Initializing => Normal
default     14m         Normal   Transition   autopilotrule/demo-volume-resize   rule: demo-volume-resize:pvc-b253ea76-8032-4f36-b5ea-fdc2085534f1 transition from Initializing => Normal
default     4m39s       Normal   Transition   autopilotrule/demo-volume-resize   rule: demo-volume-resize:pvc-aa931830-f795-43d2-9fa0-9bd74338a043 transition from Normal => Triggered
default     4m1s        Normal   Transition   autopilotrule/demo-volume-resize   rule: demo-volume-resize:pvc-aa931830-f795-43d2-9fa0-9bd74338a043 transition from Triggered => ActiveActionsPending
default     3m57s       Normal   Transition   autopilotrule/demo-volume-resize   rule: demo-volume-resize:pvc-aa931830-f795-43d2-9fa0-9bd74338a043 transition from ActiveActionsPending => ActiveActionsInProgress
default     3m52s       Normal   Transition   autopilotrule/demo-volume-resize   rule: demo-volume-resize:pvc-b253ea76-8032-4f36-b5ea-fdc2085534f1 transition from Normal => Triggered
default     3m51s       Normal   Transition   autopilotrule/demo-volume-resize   rule: demo-volume-resize:pvc-aa931830-f795-43d2-9fa0-9bd74338a043 transition from ActiveActionsInProgress => ActiveActionsTaken
default     2m56s       Normal   Transition   autopilotrule/demo-volume-resize   (combined from similar events): rule: demo-volume-resize:pvc-b253ea76-8032-4f36-b5ea-fdc2085534f1 transition from ActiveActionsInProgress => ActiveActionsTaken
```

>Note: Storage Class must have allowVolumeExpansion: true . AutoPilot will resize each time volume when volume will be utilized 80% ultil max value mentioned in autopilot rule maxsize: "100Gi"