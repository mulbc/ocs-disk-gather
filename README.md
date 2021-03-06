# ocs-disk-gather
Gather disk information from OCP4 nodes for OCS 4 local storage setup 


- the yaml creates 
   - a namespace called `ocs-disk-gatherer`
   - a service account called `ocs-disk-gatherer-sa`
   - a scc called `ocs-disk-gatherer-sa` to be attached to the service account
   - a pull secret to fetch the RHEL 8 ubi image from the registry 
     you need to get a pull secret from https://access.redhat.com/terms-based-registry/ and add it to the ubi-pull-secret section in the `ocs-disk-gatherer.yaml`
   - a daemon set called `ocs-disk-gatherer` that automatically deploys on every node labeld with `cluster.ocs.openshift.io/openshift-storage:""`
     The ds will start a loop refreshing all 10 minutes the disks it found, output to look like this:
~~~
# oc get ds -n ocs-disk-gatherer -o wide
NAME                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                                 AGE     CONTAINERS   IMAGES                        SELECTOR
ocs-disk-gatherer   3         3         3       3            3           cluster.ocs.openshift.io/openshift-storage=   2m56s   collector    registry.redhat.io/ubi8/ubi   name=ocs-disk-gatherer
#

# oc get po -o wide
NAME                      READY   STATUS    RESTARTS   AGE     IP            NODE                              NOMINATED NODE   READINESS GATES
ocs-disk-gatherer-hnm64   1/1     Running   0          3m16s   10.129.4.51   cluster-p6nss-workerocs-0-x2q2f   <none>           <none>
ocs-disk-gatherer-kjwt8   1/1     Running   0          3m16s   10.131.2.47   cluster-p6nss-workerocs-0-6ptnt   <none>           <none>
ocs-disk-gatherer-xx9ms   1/1     Running   0          3m16s   10.128.4.61   cluster-p6nss-workerocs-0-s5b42   <none>           <none>
#

# oc logs ocs-disk-gatherer-kjwt8
# NODE:cluster-p6nss-workerocs-0-6ptnt
sda : 130G : /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_60428f28-67d0-4429-9

sdb : 20G : /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_908b47fe-308a-4dfd-8

sdc : 1.2T : /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_96fd0e15-bb5f-4bc7-b

#
~~~
 
 - so run `wget https://raw.githubusercontent.com/dmoessne/ocs-disk-gather/master/ocs-disk-gatherer.yaml` modify the secret with your token and the run `oc create -f ocs-disk-gatherer.yaml`
 
There might be times when pull with secret and or tag is not possible and hence the following is provided for concenience, but mind:
  - this registry will eventually go away, so everyone is encouraged to use the term based registry and secrets
  - using sha is discouraged as well as one needs to manually keep up with current images, i.e. check if a newer version is available to avoild potential security issues

  - `oc create -f https://raw.githubusercontent.com/dmoessne/ocs-disk-gather/master/ocs-disk-gatherer-sha-no-secret.yaml` should work then 
