---
title: "볼륨"
content_template: templates/concept
weight: 10
---

{{% capture overview %}}

컨테이너의 On-disk 파일은 일시적이므로, 컨테이너에서 실행될 때 몇몇 프로그램에서 종종 문제가 발생한다. 먼저, 컨테이너가 충돌하면, kubelet은 다시 시작하지만, 파일은 손실되고, 컨테이너는 깨끗한 상태로 재시작된다. 둘째, `Pod` 에서 컨테이너를 함께 실행할 때, 해당 컨테이너간에 파일을 공유해야 하는 경우가 종종 있다. Kubernetes `Volume` 추상화는 이러한 두 가지 문제를 모두 해결한다.

[Pods](/docs/user-guide/pods) 문서를 참고하라.

{{% /capture %}}


{{% capture body %}}

## Background

Docker에는 [볼륨](https://docs.docker.com/engine/admin/volumes/) 개념이 있지만 다소 느슨하고 덜 관리된다. Docker에서 볼륨은 단순히 디스크 또는 다른 컨테이너의 디렉토리이다. 라이프타임는 관리되지 않고 최근까지는 로컬 디스크 백업 볼륨만 있었다. Docker는 이제 볼륨 드라이버를 제공하지만 기능은 매우 제한적이다. (예를 들어, Docker 1.7부터 컨테이너당 하나의 볼륨 드라이버만 허용되며, 매개 변수를 볼륨에 전달할 수 있는 방법이 없다.)

반면, Kubernetes 볼륨은 볼륨을 묶는 파드와 동일하게 명시적인 라이프타임을 갖습니다. 결과적으로, 볼륨은 파드 내에서 실행되는 모든 컨테이너보다 수명이 길고 컨테이너를 다시 시작해도 데이터가 보존된다. 물론, 파드가 존재하지 않으면 볼륨도 존재하지 않는다. 이보다 더 중요한 것은 Kubernetes는 많은 유형의 볼륨을 지원하며, 파드는 여러 볼륨을 동시에 사용할 수 있다.

기본적으로, 볼륨은 디렉토리일 뿐이며 일부 데이터가 있을 수 있고, 파드의 컨테이너에서 액세스 할 수 있다. 해당 디렉토리의 방법, 디렉토리를 지원하는 매체 및 디렉토리의 내용은 사용된 특정 볼륨 유형에 따라 결정된다.

볼륨을 사용하기 위해, 파드는 파드에 제공할 볼륨(`.spec.volumes` 필드)과 컨테이너에 마운트 할 위치(`.spec.containers.volumeMounts` 필드)를 지정한다.

컨테이너의 프로세스는 Docker 이미지와 볼륨으로 구성된 파일시스템 뷰를 본다. [Docker 이미지](https://docs.docker.com/userguide/dockerimages/)는 파일시스템 계층의 루트에 있으며, 모든 볼륨은 이미지 내의 지정된 경로에 마운트된다. 볼륨은 다른 볼륨에 마운트 할 수 없거나 다른 볼륨에 대한 하드 링크를 가질 수 없다. 파드의 각 컨테이너는 각 볼륨을 마운트 할 위치를 독립적으로 지정해야 한다.

## 볼륨의 종류

쿠버네티스에서 지원하는 몇몇 타입의 볼륨들:

   * [awsElasticBlockStore](#awselasticblockstore)
   * [azureDisk](#azuredisk)
   * [azureFile](#azurefile)
   * [cephfs](#cephfs)
   * [cinder](#cinder)
   * [configMap](#configmap)
   * [csi](#csi)
   * [downwardAPI](#downwardapi)
   * [emptyDir](#emptydir)
   * [fc (fibre channel)](#fc)
   * [flexVolume](#flexVolume)
   * [flocker](#flocker)
   * [gcePersistentDisk](#gcepersistentdisk)
   * [gitRepo (deprecated)](#gitrepo)
   * [glusterfs](#glusterfs)
   * [hostPath](#hostpath)
   * [iscsi](#iscsi)
   * [local](#local)
   * [nfs](#nfs)
   * [persistentVolumeClaim](#persistentvolumeclaim)
   * [projected](#projected)
   * [portworxVolume](#portworxvolume)
   * [quobyte](#quobyte)
   * [rbd](#rbd)
   * [scaleIO](#scaleio)
   * [secret](#secret)
   * [storageos](#storageos)
   * [vsphereVolume](#vspherevolume)

컨트리뷰션의 추가를 환영한다.

### awsElasticBlockStore {#awselasticblockstore}

awsElasticBlockStore 볼륨은 Amazon Web Services (AWS) [EBS 볼륨](http://aws.amazon.com/ebs/)을 파드에 마운트한다. 파드를 제거 할 때 지워지는 emptyDir과 달리 EBS 볼륨의 내용은 유지되고, 볼륨은 마운트 해제된다. 즉, EBS 볼륨에 데이터를 미리 채울 수 있으며, 파드 간에 데이터를 "핸드 오프" 할 수 있다.

{{< caution >}}
사용하기 전에 `aws ec2 create-volume` 또는 AWS API를 사용하여 EBS 볼륨을 생성해야 한다.
{{< /caution >}}

`awsElasticBlockStore` 볼륨을 사용할 때 몇 가지 제한 사항이 있다:

* 파드가 실행중인 노드는 AWS EC2 인스턴스여야 한다.
* 해당 인스턴스는 EBS 볼륨과 동일한 리전 및 가용성-zone 영역에 있어야 한다.
* EBS는 볼륨을 마운트하는 단일 EC2 인스턴스 만 지원한다.

#### EBS 볼륨 생성

파드와 함께 EBS 볼륨을 사용하려면, 먼저 생성해야 한다.

```shell
aws ec2 create-volume --availability-zone=eu-west-1a --size=10 --volume-type=gp2
```

zone 영역이 클러스터를 가져온 zone 영역과 일치하는지 확인하라. (또한 사이즈 및 EBS 볼륨 유형이 사용에 적합한지 확인하라!)

#### AWS EBS 예제 구성

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-ebs
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-ebs
      name: test-volume
  volumes:
  - name: test-volume
    # This AWS EBS volume must already exist.
    awsElasticBlockStore:
      volumeID: <volume-id>
      fsType: ext4
```

#### CSI 마이그레이션

{{< feature-state for_k8s_version="v1.14" state="alpha" >}}

The CSI Migration feature for awsElasticBlockStore, when enabled, shims all plugin operations from the existing in-tree plugin to the `ebs.csi.aws.com` Container Storage Interface (CSI) Driver. 
In order to use this feature, the [AWS EBS CSI Driver](https://github.com/kubernetes-sigs/aws-ebs-csi-driver) must be installed on the cluster and the `CSIMigration` and `CSIMigrationAWS` Alpha features must be enabled.

### azureDisk {#azuredisk}

A `azureDisk` is used to mount a Microsoft Azure [Data Disk](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-about-disks-vhds/) into a Pod.

More details can be found [here](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/azure_disk/README.md).

#### CSI Migration

{{< feature-state for_k8s_version="v1.15" state="alpha" >}}

The CSI Migration feature for azureDisk, when enabled, shims all plugin operations
from the existing in-tree plugin to the `disk.csi.azure.com` Container
Storage Interface (CSI) Driver. In order to use this feature, the [Azure Disk CSI
Driver](https://github.com/kubernetes-sigs/azuredisk-csi-driver)
must be installed on the cluster and the `CSIMigration` and `CSIMigrationAzureDisk`
Alpha features must be enabled.

### azureFile {#azurefile}

A `azureFile` is used to mount a Microsoft Azure File Volume (SMB 2.1 and 3.0)
into a Pod.

More details can be found [here](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/azure_file/README.md).

#### CSI Migration

{{< feature-state for_k8s_version="v1.15" state="alpha" >}}

The CSI Migration feature for azureFile, when enabled, shims all plugin operations
from the existing in-tree plugin to the `file.csi.azure.com` Container
Storage Interface (CSI) Driver. In order to use this feature, the [Azure File CSI
Driver](https://github.com/kubernetes-sigs/azurefile-csi-driver)
must be installed on the cluster and the `CSIMigration` and `CSIMigrationAzureFile`
Alpha features must be enabled.

### cephfs {#cephfs}

A `cephfs` volume allows an existing CephFS volume to be
mounted into your Pod. Unlike `emptyDir`, which is erased when a Pod is
removed, the contents of a `cephfs` volume are preserved and the volume is merely
unmounted.  This means that a CephFS volume can be pre-populated with data, and
that data can be "handed off" between Pods.  CephFS can be mounted by multiple
writers simultaneously.

{{< caution >}}
You must have your own Ceph server running with the share exported before you can use it.
{{< /caution >}}

See the [CephFS example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/volumes/cephfs/) for more details.

### cinder {#cinder}

{{< note >}}
Prerequisite: Kubernetes with OpenStack Cloud Provider configured. For cloudprovider
configuration please refer [cloud provider openstack](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/#openstack).
{{< /note >}}

`cinder` is used to mount OpenStack Cinder Volume into your Pod.

#### Cinder Volume Example configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-cinder
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-cinder-container
    volumeMounts:
    - mountPath: /test-cinder
      name: test-volume
  volumes:
  - name: test-volume
    # This OpenStack volume must already exist.
    cinder:
      volumeID: <volume-id>
      fsType: ext4
```

#### CSI Migration

{{< feature-state for_k8s_version="v1.14" state="alpha" >}}

The CSI Migration feature for Cinder, when enabled, shims all plugin operations
from the existing in-tree plugin to the `cinder.csi.openstack.org` Container
Storage Interface (CSI) Driver. In order to use this feature, the [Openstack Cinder CSI
Driver](https://github.com/kubernetes/cloud-provider-openstack/blob/master/docs/using-cinder-csi-plugin.md)
must be installed on the cluster and the `CSIMigration` and `CSIMigrationOpenStack`
Alpha features must be enabled.

### configMap {#configmap}

[`configMap`](/docs/tasks/configure-pod-container/configure-pod-configmap/) 리소스는 구성 데이터를 파드에 주입하는 방법을 제공한다. `ConfigMap` 객체에 저장된 데이터는 `configMap` 유형의 볼륨에서 참조된 다음 파드에서 실행되는 컨테이너화된 응용 프로그램에 의해 사용될 수 있다.

`configMap` 오브젝트를 참조 할 때, 볼륨에 이름을 제공하여 참조 할 수 있다. ConfigMap의 특정 항목에 사용할 경로를 사용자 정의할 수도 있다.
예를 들어, `log-config` ConfigMap을 `configmap-pod`라는 파드에 마운트하려면 아래 YAML을 사용하면 된다:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: test
      image: busybox
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: log-config
        items:
          - key: log_level
            path: log_level
```

The `log-config` ConfigMap is mounted as a volume, and all contents stored in its `log_level` entry are mounted into the Pod at path "`/etc/config/log_level`".
Note that this path is derived from the volume's `mountPath` and the `path` keyed with `log_level`.

{{< caution >}}
You must create a [ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/) before you can use it.
{{< /caution >}}

{{< note >}}
A Container using a ConfigMap as a [subPath](#using-subpath) volume mount will not
receive ConfigMap updates.
{{< /note >}}

### downwardAPI {#downwardapi}

A `downwardAPI` volume is used to make downward API data available to applications.
It mounts a directory and writes the requested data in plain text files.

{{< note >}}
A Container using Downward API as a [subPath](#using-subpath) volume mount will not
receive Downward API updates.
{{< /note >}}

See the [`downwardAPI` volume example](/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)  for more details.

### emptyDir {#emptydir}

`emptyDir` 볼륨은 파드가 노드에 할당될 때 처음 생성되며, 해당 노드에서 파드가 실행되는 동안 존재한다. 이름에서 알 수 있듯이 처음에는 비어 있습니다. 파드의 컨테이너는 모두 `emptyDir` 볼륨에서 동일한 파일을 읽고 쓸 수 있지만, 해당 볼륨은 각 컨테이너의 동일하거나 다른 경로에 마운트 될 수 있다.
When a Pod is removed from a node for any reason, the data in the `emptyDir` is deleted forever.

{{< note >}}
A Container crashing does *NOT* remove a Pod from a node, so the data in an `emptyDir` volume is safe across Container crashes.
{{< /note >}}

Some uses for an `emptyDir` are:

* scratch space, such as for a disk-based merge sort
* checkpointing a long computation for recovery from crashes
* holding files that a content-manager Container fetches while a webserver
  Container serves the data

By default, `emptyDir` volumes are stored on whatever medium is backing the
node - that might be disk or SSD or network storage, depending on your
environment.  However, you can set the `emptyDir.medium` field to `"Memory"`
to tell Kubernetes to mount a tmpfs (RAM-backed filesystem) for you instead.
While tmpfs is very fast, be aware that unlike disks, tmpfs is cleared on
node reboot and any files you write will count against your Container's
memory limit.

#### Example Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

### fc (fibre channel) {#fc}

An `fc` volume allows an existing fibre channel volume to be mounted in a Pod.
You can specify single or multiple target World Wide Names using the parameter
`targetWWNs` in your volume configuration. If multiple WWNs are specified,
targetWWNs expect that those WWNs are from multi-path connections.

{{< caution >}}
You must configure FC SAN Zoning to allocate and mask those LUNs (volumes) to the target WWNs beforehand so that Kubernetes hosts can access them.
{{< /caution >}}

See the [FC example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/fibre_channel) for more details.

### flocker {#flocker}

[Flocker](https://github.com/ClusterHQ/flocker) is an open-source clustered Container data volume manager. It provides management
and orchestration of data volumes backed by a variety of storage backends.

A `flocker` volume allows a Flocker dataset to be mounted into a Pod. If the
dataset does not already exist in Flocker, it needs to be first created with the Flocker
CLI or by using the Flocker API. If the dataset already exists it will be
reattached by Flocker to the node that the Pod is scheduled. This means data
can be "handed off" between Pods as required.

{{< caution >}}
You must have your own Flocker installation running before you can use it.
{{< /caution >}}

See the [Flocker example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/flocker) for more details.

### gcePersistentDisk {#gcepersistentdisk}

A `gcePersistentDisk` volume mounts a Google Compute Engine (GCE) [Persistent
Disk](http://cloud.google.com/compute/docs/disks) into your Pod.  Unlike
`emptyDir`, which is erased when a Pod is removed, the contents of a PD are
preserved and the volume is merely unmounted.  This means that a PD can be
pre-populated with data, and that data can be "handed off" between Pods.

{{< caution >}}
You must create a PD using `gcloud` or the GCE API or UI before you can use it.
{{< /caution >}}

There are some restrictions when using a `gcePersistentDisk`:

* the nodes on which Pods are running must be GCE VMs
* those VMs need to be in the same GCE project and zone as the PD

A feature of PD is that they can be mounted as read-only by multiple consumers
simultaneously.  This means that you can pre-populate a PD with your dataset
and then serve it in parallel from as many Pods as you need.  Unfortunately,
PDs can only be mounted by a single consumer in read-write mode - no
simultaneous writers allowed.

Using a PD on a Pod controlled by a ReplicationController will fail unless
the PD is read-only or the replica count is 0 or 1.

#### Creating a PD

Before you can use a GCE PD with a Pod, you need to create it.

```shell
gcloud compute disks create --size=500GB --zone=us-central1-a my-data-disk
```

#### Example Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    # This GCE PD must already exist.
    gcePersistentDisk:
      pdName: my-data-disk
      fsType: ext4
```

#### Regional Persistent Disks
{{< feature-state for_k8s_version="v1.10" state="beta" >}}

The [Regional Persistent Disks](https://cloud.google.com/compute/docs/disks/#repds) feature allows the creation of Persistent Disks that are available in two zones within the same region. In order to use this feature, the volume must be provisioned as a PersistentVolume; referencing the volume directly from a pod is not supported.

#### Manually provisioning a Regional PD PersistentVolume
Dynamic provisioning is possible using a [StorageClass for GCE PD](/docs/concepts/storage/storage-classes/#gce).
Before creating a PersistentVolume, you must create the PD:
```shell
gcloud beta compute disks create --size=500GB my-data-disk
    --region us-central1
    --replica-zones us-central1-a,us-central1-b
```
Example PersistentVolume spec:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-volume
  labels:
    failure-domain.beta.kubernetes.io/zone: us-central1-a__us-central1-b
spec:
  capacity:
    storage: 400Gi
  accessModes:
  - ReadWriteOnce
  gcePersistentDisk:
    pdName: my-data-disk
    fsType: ext4
```

#### CSI Migration

{{< feature-state for_k8s_version="v1.14" state="alpha" >}}

The CSI Migration feature for GCE PD, when enabled, shims all plugin operations
from the existing in-tree plugin to the `pd.csi.storage.gke.io` Container
Storage Interface (CSI) Driver. In order to use this feature, the [GCE PD CSI
Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver)
must be installed on the cluster and the `CSIMigration` and `CSIMigrationGCE`
Alpha features must be enabled.

### gitRepo (deprecated) {#gitrepo}

{{< warning >}}
The gitRepo volume type is deprecated. To provision a container with a git repo, mount an [EmptyDir](#emptydir) into an InitContainer that clones the repo using git, then mount the [EmptyDir](#emptydir) into the Pod's container.
{{< /warning >}}

A `gitRepo` volume is an example of what can be done as a volume plugin.  It
mounts an empty directory and clones a git repository into it for your Pod to
use.  In the future, such volumes may be moved to an even more decoupled model,
rather than extending the Kubernetes API for every such use case.

Here is an example of gitRepo volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: server
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - mountPath: /mypath
      name: git-volume
  volumes:
  - name: git-volume
    gitRepo:
      repository: "git@somewhere:me/my-git-repository.git"
      revision: "22f1d8406d464b0c0874075539c1f2e96c253775"
```

### glusterfs {#glusterfs}

A `glusterfs` volume allows a [Glusterfs](http://www.gluster.org) (an open
source networked filesystem) volume to be mounted into your Pod.  Unlike
`emptyDir`, which is erased when a Pod is removed, the contents of a
`glusterfs` volume are preserved and the volume is merely unmounted.  This
means that a glusterfs volume can be pre-populated with data, and that data can
be "handed off" between Pods.  GlusterFS can be mounted by multiple writers
simultaneously.

{{< caution >}}
You must have your own GlusterFS installation running before you can use it.
{{< /caution >}}

See the [GlusterFS example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/volumes/glusterfs) for more details.

### hostPath {#hostpath}

`hostPath` 볼륨은 파일 또는 디렉토리를 호스트 노드의 파일 시스템에서 파드로 마운트한다. 이것은 대부분의 파드에 필요한 것은 아니지만, 일부 응용 프로그램에는 강력한 탈출구를 제공한다.

예를 들어, `hostPath`의 일부 용도는:

* Docker 내부에 액세스해야 하는 컨테이너 실행; `/var/lib/docker`의 `hostPath`를 사용한다.
* 컨테이너에서 cAdvisor 실행; `/sys`의 `hostPath`를 사용한다.
* 파드가 실행하기 전에 주어진 `hostPath`가 존재해야 하는지, 생성해야 하는지, 아니면 존재하는 것을 지정하는지 설정한다.

필수 `path` 속성 외에도 사용자는 선택적으로 `hostPath` 볼륨의 유형을 지정할 수 있다.

`type` 필드에 지원되는 값은:

| Value | Behavior |
|:------|:---------|
| | 빈 문자열(기본값)은 이전 버전과의 호환성을 위한 것이므로, hostPath 볼륨을 마운트하기 전에 검사가 수행되지 않는다. |
| `DirectoryOrCreate` | If nothing exists at the given path, an empty directory will be created there as needed with permission set to 0755, having the same group and ownership with Kubelet. |
| `Directory` | A directory must exist at the given path |
| `FileOrCreate` | If nothing exists at the given path, an empty file will be created there as needed with permission set to 0644, having the same group and ownership with Kubelet. |
| `File` | A file must exist at the given path |
| `Socket` | A UNIX socket must exist at the given path |
| `CharDevice` | A character device must exist at the given path |
| `BlockDevice` | A block device must exist at the given path |

Watch out when using this type of volume, because:

* Pods with identical configuration (such as created from a podTemplate) may
  behave differently on different nodes due to different files on the nodes
* when Kubernetes adds resource-aware scheduling, as is planned, it will not be
  able to account for resources used by a `hostPath`
* the files or directories created on the underlying hosts are only writable by root. You
  either need to run your process as root in a
  [privileged Container](/docs/user-guide/security-context) or modify the file
  permissions on the host to be able to write to a `hostPath` volume

#### Example Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: Directory
```

### iscsi {#iscsi}

An `iscsi` volume allows an existing iSCSI (SCSI over IP) volume to be mounted
into your Pod.  Unlike `emptyDir`, which is erased when a Pod is removed, the
contents of an `iscsi` volume are preserved and the volume is merely
unmounted.  This means that an iscsi volume can be pre-populated with data, and
that data can be "handed off" between Pods.

{{< caution >}}
You must have your own iSCSI server running with the volume created before you can use it.
{{< /caution >}}

A feature of iSCSI is that it can be mounted as read-only by multiple consumers
simultaneously.  This means that you can pre-populate a volume with your dataset
and then serve it in parallel from as many Pods as you need.  Unfortunately,
iSCSI volumes can only be mounted by a single consumer in read-write mode - no
simultaneous writers allowed.

See the [iSCSI example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/volumes/iscsi) for more details.

### local {#local}

{{< feature-state for_k8s_version="v1.14" state="stable" >}}

`로컬` 볼륨은 디스크, 파티션 또는 디렉토리와 같은 마운트된 로컬 저장 장치를 나타낸다.

로컬 볼륨은 정적으로 생성된 PersistentVolume으로만 사용할 수 있다. 동적 프로비저닝은 아직 지원하지 않는다.

`hostPath` 볼륨과 비교해서, 로컬 볼륨은 
Compared to `hostPath` volumes, local volumes can be used in a durable and portable manner without manually scheduling Pods to nodes, as the system is aware of the volume's node constraints by looking at the node affinity on the PersistentVolume.

However, local volumes are still subject to the availability of the underlying node and are not suitable for all applications. 
If a node becomes unhealthy, then the local volume will also become inaccessible, and a Pod using it will not be able to run. 
Applications using local volumes must be able to tolerate this reduced availability, as well as potential data loss, depending on the durability characteristics of the underlying disk.

The following is an example of PersistentVolume spec using a `local` volume and
`nodeAffinity`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  # volumeMode field requires BlockVolume Alpha feature gate to be enabled.
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - example-node
```

PersistentVolume `nodeAffinity` is required when using local volumes. 
It enables the Kubernetes scheduler to correctly schedule Pods using local volumes to the correct node.

PersistentVolume `volumeMode` can now be set to "Block" (instead of the default
value "Filesystem") to expose the local volume as a raw block device. The
`volumeMode` field requires `BlockVolume` Alpha feature gate to be enabled.

When using local volumes, it is recommended to create a StorageClass with
`volumeBindingMode` set to `WaitForFirstConsumer`. See the
[example](/docs/concepts/storage/storage-classes/#local). Delaying volume binding ensures
that the PersistentVolumeClaim binding decision will also be evaluated with any
other node constraints the Pod may have, such as node resource requirements, node
selectors, Pod affinity, and Pod anti-affinity.

An external static provisioner can be run separately for improved management of
the local volume lifecycle. Note that this provisioner does not support dynamic
provisioning yet. For an example on how to run an external local provisioner,
see the [local volume provisioner user
guide](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner).

{{< note >}}
The local PersistentVolume requires manual cleanup and deletion by the
user if the external static provisioner is not used to manage the volume
lifecycle.
{{< /note >}}

### nfs {#nfs}

An `nfs` volume allows an existing NFS (Network File System) share to be
mounted into your Pod. Unlike `emptyDir`, which is erased when a Pod is
removed, the contents of an `nfs` volume are preserved and the volume is merely
unmounted.  This means that an NFS volume can be pre-populated with data, and
that data can be "handed off" between Pods.  NFS can be mounted by multiple
writers simultaneously.

{{< caution >}}
You must have your own NFS server running with the share exported before you can use it.
{{< /caution >}}

See the [NFS example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/nfs) for more details.

### persistentVolumeClaim {#persistentvolumeclaim}

[PersistentVolume](/docs/concepts/storage/persistent-volumes/)을 파드에 마운트하는데 `persistentVolumeClaim` 볼륨이 사용된다. PersistentVolume은 사용자가 특정 클라우드 환경의 세부사항을 몰라도 영속성 있는 스토리지(GCE PersistentDisk거나 iSCSI 볼륨처럼)를 "클레임" 할 수 있는 방법이다.

자세한 내용은 [PersistentVolumes 예제](/docs/concepts/storage/persistent-volumes/)를 참조하세요.

### projected {#projected}

`프로젝션` 볼륨은 여러 기존 볼륨 소스랑 동일한 디렉토리에 매핑한다.

Currently, the following types of volume sources can be projected:

- [`secret`](#secret)
- [`downwardAPI`](#downwardapi)
- [`configMap`](#configmap)
- `serviceAccountToken`

모든 소스는 파드와 동일한 네임스페이스에 있어야 한다. 자세한 내용은 [올인원 볼륨 디자인 문서](https://github.com/kubernetes/community/blob/{{< param "githubbranch" >}}/contributors/design-proposals/node/all-in-one-volume.md)를 참조하세요.

서비스 계정 토큰의 프로젝션은 Kubernetes 1.11 에서 소개된 기능이고 1.12 Beta 로 승격되었다. 1.11 버전에서 이 기능을 사용하려면, `TokenRequestProjection` [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) 를 True로 설정해야 한다.

#### secret, downward API와 configmap 이 있는 파드 YAML 예제.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  containers:
  - name: container-test
    image: busybox
    volumeMounts:
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: mysecret
          items:
            - key: username
              path: my-group/my-username
      - downwardAPI:
          items:
            - path: "labels"
              fieldRef:
                fieldPath: metadata.labels
            - path: "cpu_limit"
              resourceFieldRef:
                containerName: container-test
                resource: limits.cpu
      - configMap:
          name: myconfigmap
          items:
            - key: config
              path: my-group/my-config
```

#### 기본이 아닌 권한 모드가 설정된 여러 secret 을 가진 파드의 YAML 예제.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  containers:
  - name: container-test
    image: busybox
    volumeMounts:
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: mysecret
          items:
            - key: username
              path: my-group/my-username
      - secret:
          name: mysecret2
          items:
            - key: password
              path: my-group/my-password
              mode: 511
```

Each projected volume source is listed in the spec under `sources`. 
The parameters are nearly the same with two exceptions:

* For secrets, the `secretName` field has been changed to `name` to be consistent
  with ConfigMap naming.
* The `defaultMode` can only be specified at the projected level and not for each
  volume source. However, as illustrated above, you can explicitly set the `mode`
  for each individual projection.

When the `TokenRequestProjection` feature is enabled, you can inject the token
for the current [service account](/docs/reference/access-authn-authz/authentication/#service-account-tokens)
into a Pod at a specified path. Below is an example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-token-test
spec:
  containers:
  - name: container-test
    image: busybox
    volumeMounts:
    - name: token-vol
      mountPath: "/service-account"
      readOnly: true
  volumes:
  - name: token-vol
    projected:
      sources:
      - serviceAccountToken:
          audience: api
          expirationSeconds: 3600
          path: token
```

The example Pod has a projected volume containing the injected service account
token. This token can be used by Pod containers to access the Kubernetes API
server, for example. The `audience` field contains the intended audience of the
token. A recipient of the token must identify itself with an identifier specified
in the audience of the token, and otherwise should reject the token. This field
is optional and it defaults to the identifier of the API server.

The `expirationSeconds` is the expected duration of validity of the service account
token. It defaults to 1 hour and must be at least 10 minutes (600 seconds). An administrator
can also limit its maximum value by specifying the `--service-account-max-token-expiration`
option for the API server. The `path` field specifies a relative path to the mount point
of the projected volume.

{{< note >}}
A Container using a projected volume source as a [subPath](#using-subpath) volume mount will not
receive updates for those volume sources.
{{< /note >}}

### portworxVolume {#portworxvolume}

A `portworxVolume` is an elastic block storage layer that runs hyperconverged with
Kubernetes. [Portworx](https://portworx.com/use-case/kubernetes-storage/) fingerprints storage in a server, tiers based on capabilities,
and aggregates capacity across multiple servers. Portworx runs in-guest in virtual machines or on bare metal Linux nodes.

A `portworxVolume` can be dynamically created through Kubernetes or it can also
be pre-provisioned and referenced inside a Kubernetes Pod.
Here is an example Pod referencing a pre-provisioned PortworxVolume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-portworx-volume-pod
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /mnt
      name: pxvol
  volumes:
  - name: pxvol
    # This Portworx volume must already exist.
    portworxVolume:
      volumeID: "pxvol"
      fsType: "<fs-type>"
```

{{< caution >}}
Make sure you have an existing PortworxVolume with name `pxvol`
before using it in the Pod.
{{< /caution >}}

More details and examples can be found [here](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/portworx/README.md).

### quobyte {#quobyte}

A `quobyte` volume allows an existing [Quobyte](http://www.quobyte.com) volume to
be mounted into your Pod.

{{< caution >}}
You must have your own Quobyte setup running with the volumes
created before you can use it.
{{< /caution >}}

Quobyte supports the {{< glossary_tooltip text="Container Storage Interface" term_id="csi" >}}.
CSI is the recommended plugin to use Quobyte volumes inside Kubernetes. Quobyte's
GitHub project has [instructions](https://github.com/quobyte/quobyte-csi#quobyte-csi) for deploying Quobyte using CSI, along with examples.

### rbd {#rbd}

An `rbd` volume allows a [Rados Block
Device](http://ceph.com/docs/master/rbd/rbd/) volume to be mounted into your
Pod.  Unlike `emptyDir`, which is erased when a Pod is removed, the contents of
a `rbd` volume are preserved and the volume is merely unmounted.  This
means that a RBD volume can be pre-populated with data, and that data can
be "handed off" between Pods.

{{< caution >}}
You must have your own Ceph installation running before you can use RBD.
{{< /caution >}}

A feature of RBD is that it can be mounted as read-only by multiple consumers
simultaneously.  This means that you can pre-populate a volume with your dataset
and then serve it in parallel from as many Pods as you need.  Unfortunately,
RBD volumes can only be mounted by a single consumer in read-write mode - no
simultaneous writers allowed.

See the [RBD example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/volumes/rbd) for more details.

### scaleIO {#scaleio}

ScaleIO is a software-based storage platform that can use existing hardware to
create clusters of scalable shared block networked storage. The `scaleIO` volume
plugin allows deployed Pods to access existing ScaleIO
volumes (or it can dynamically provision new volumes for persistent volume claims, see
[ScaleIO Persistent Volumes](/docs/concepts/storage/persistent-volumes/#scaleio)).

{{< caution >}}
You must have an existing ScaleIO cluster already setup and
running with the volumes created before you can use them.
{{< /caution >}}

The following is an example of Pod configuration with ScaleIO:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-0
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: pod-0
    volumeMounts:
    - mountPath: /test-pd
      name: vol-0
  volumes:
  - name: vol-0
    scaleIO:
      gateway: https://localhost:443/api
      system: scaleio
      protectionDomain: sd0
      storagePool: sp1
      volumeName: vol-0
      secretRef:
        name: sio-secret
      fsType: xfs
```

For further detail, please see the [ScaleIO examples](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/scaleio).

### secret {#secret}

A `secret` volume is used to pass sensitive information, such as passwords, to
Pods.  You can store secrets in the Kubernetes API and mount them as files for
use by Pods without coupling to Kubernetes directly.  `secret` volumes are
backed by tmpfs (a RAM-backed filesystem) so they are never written to
non-volatile storage.

{{< caution >}}
You must create a secret in the Kubernetes API before you can use it.
{{< /caution >}}

{{< note >}}
A Container using a Secret as a [subPath](#using-subpath) volume mount will not
receive Secret updates.
{{< /note >}}

Secrets are described in more detail [here](/docs/user-guide/secrets).

### storageOS {#storageos}

A `storageos` volume allows an existing [StorageOS](https://www.storageos.com)
volume to be mounted into your Pod.

StorageOS runs as a Container within your Kubernetes environment, making local
or attached storage accessible from any node within the Kubernetes cluster.
Data can be replicated to protect against node failure. Thin provisioning and
compression can improve utilization and reduce cost.

At its core, StorageOS provides block storage to Containers, accessible via a file system.

The StorageOS Container requires 64-bit Linux and has no additional dependencies.
A free developer license is available.

{{< caution >}}
You must run the StorageOS Container on each node that wants to
access StorageOS volumes or that will contribute storage capacity to the pool.
For installation instructions, consult the
[StorageOS documentation](https://docs.storageos.com).
{{< /caution >}}

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: redis
    role: master
  name: test-storageos-redis
spec:
  containers:
    - name: master
      image: kubernetes/redis:v1
      env:
        - name: MASTER
          value: "true"
      ports:
        - containerPort: 6379
      volumeMounts:
        - mountPath: /redis-master-data
          name: redis-data
  volumes:
    - name: redis-data
      storageos:
        # The `redis-vol01` volume must already exist within StorageOS in the `default` namespace.
        volumeName: redis-vol01
        fsType: ext4
```

For more information including Dynamic Provisioning and Persistent Volume Claims, please see the
[StorageOS examples](https://github.com/kubernetes/examples/blob/master/volumes/storageos).

### vsphereVolume {#vspherevolume}

{{< note >}}
Prerequisite: Kubernetes with vSphere Cloud Provider configured. For cloudprovider
configuration please refer [vSphere getting started guide](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/).
{{< /note >}}

A `vsphereVolume` is used to mount a vSphere VMDK Volume into your Pod.  The contents
of a volume are preserved when it is unmounted. It supports both VMFS and VSAN datastore.

{{< caution >}}
You must create VMDK using one of the following methods before using with Pod.
{{< /caution >}}

#### Creating a VMDK volume

Choose one of the following methods to create a VMDK.

{{< tabs name="tabs_volumes" >}}
{{% tab name="Create using vmkfstools" %}}
First ssh into ESX, then use the following command to create a VMDK:

```shell
vmkfstools -c 2G /vmfs/volumes/DatastoreName/volumes/myDisk.vmdk
```
{{% /tab %}}
{{% tab name="Create using vmware-vdiskmanager" %}}
Use the following command to create a VMDK:

```shell
vmware-vdiskmanager -c -t 0 -s 40GB -a lsilogic myDisk.vmdk
```
{{% /tab %}}

{{< /tabs >}}


#### vSphere VMDK Example configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-vmdk
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-vmdk
      name: test-volume
  volumes:
  - name: test-volume
    # This VMDK volume must already exist.
    vsphereVolume:
      volumePath: "[DatastoreName] volumes/myDisk"
      fsType: ext4
```

More examples can be found [here](https://github.com/kubernetes/examples/tree/master/staging/volumes/vsphere).


## Using subPath

Sometimes, it is useful to share one volume for multiple uses in a single Pod. The `volumeMounts.subPath`
property can be used to specify a sub-path inside the referenced volume instead of its root.

Here is an example of a Pod with a LAMP stack (Linux Apache Mysql PHP) using a single, shared volume.
The HTML contents are mapped to its `html` folder, and the databases will be stored in its `mysql` folder:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-lamp-site
spec:
    containers:
    - name: mysql
      image: mysql
      env:
      - name: MYSQL_ROOT_PASSWORD
        value: "rootpasswd"
      volumeMounts:
      - mountPath: /var/lib/mysql
        name: site-data
        subPath: mysql
    - name: php
      image: php:7.0-apache
      volumeMounts:
      - mountPath: /var/www/html
        name: site-data
        subPath: html
    volumes:
    - name: site-data
      persistentVolumeClaim:
        claimName: my-lamp-site-data
```

### Using subPath with expanded environment variables

{{< feature-state for_k8s_version="v1.15" state="beta" >}}


Use the `subPathExpr` field to construct `subPath` directory names from Downward API environment variables.
Before you use this feature, you must enable the `VolumeSubpathEnvExpansion` feature gate.
The `subPath` and `subPathExpr` properties are mutually exclusive.

In this example, a Pod uses `subPathExpr` to create a directory `pod1` within the hostPath volume `/var/log/pods`, using the pod name from the Downward API.  The host directory `/var/log/pods/pod1` is mounted at `/logs` in the container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container1
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
    image: busybox
    command: [ "sh", "-c", "while [ true ]; do echo 'Hello'; sleep 10; done | tee -a /logs/hello.txt" ]
    volumeMounts:
    - name: workdir1
      mountPath: /logs
      subPathExpr: $(POD_NAME)
  restartPolicy: Never
  volumes:
  - name: workdir1
    hostPath:
      path: /var/log/pods
```

## Resources

The storage media (Disk, SSD, etc.) of an `emptyDir` volume is determined by the
medium of the filesystem holding the kubelet root dir (typically
`/var/lib/kubelet`).  There is no limit on how much space an `emptyDir` or
`hostPath` volume can consume, and no isolation between Containers or between
Pods.

In the future, we expect that `emptyDir` and `hostPath` volumes will be able to
request a certain amount of space using a [resource](/docs/user-guide/compute-resources)
specification, and to select the type of media to use, for clusters that have
several media types.

## Out-of-Tree Volume Plugins
The Out-of-tree volume plugins include the Container Storage Interface (CSI)
and FlexVolume. They enable storage vendors to create custom storage plugins
without adding them to the Kubernetes repository.

Before the introduction of CSI and FlexVolume, all volume plugins (like
volume types listed above) were "in-tree" meaning they were built, linked,
compiled, and shipped with the core Kubernetes binaries and extend the core
Kubernetes API. This meant that adding a new storage system to Kubernetes (a
volume plugin) required checking code into the core Kubernetes code repository.

Both CSI and FlexVolume allow volume plugins to be developed independent of
the Kubernetes code base, and deployed (installed) on Kubernetes clusters as
extensions.

For storage vendors looking to create an out-of-tree volume plugin, please refer
to [this FAQ](https://github.com/kubernetes/community/blob/master/sig-storage/volume-plugin-faq.md).

### CSI

[Container Storage Interface](https://github.com/container-storage-interface/spec/blob/master/spec.md) (CSI)
defines a standard interface for container orchestration systems (like
Kubernetes) to expose arbitrary storage systems to their container workloads.

Please read the [CSI design proposal](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/storage/container-storage-interface.md) for more information.

CSI support was introduced as alpha in Kubernetes v1.9, moved to beta in
Kubernetes v1.10, and is GA in Kubernetes v1.13.

{{< note >}}
Support for CSI spec versions 0.2 and 0.3 are deprecated in Kubernetes
v1.13 and will be removed in a future release.
{{< /note >}}

{{< note >}}
CSI drivers may not be compatible across all Kubernetes releases.
Please check the specific CSI driver's documentation for supported
deployments steps for each Kubernetes release and a compatibility matrix.
{{< /note >}}

Once a CSI compatible volume driver is deployed on a Kubernetes cluster, users
may use the `csi` volume type to attach, mount, etc. the volumes exposed by the
CSI driver.

The `csi` volume type does not support direct reference from Pod and may only be
referenced in a Pod via a `PersistentVolumeClaim` object.

The following fields are available to storage administrators to configure a CSI
persistent volume:

- `driver`: A string value that specifies the name of the volume driver to use.
  This value must correspond to the value returned in the `GetPluginInfoResponse`
  by the CSI driver as defined in the [CSI spec](https://github.com/container-storage-interface/spec/blob/master/spec.md#getplugininfo).
  It is used by Kubernetes to identify which CSI driver to call out to, and by
  CSI driver components to identify which PV objects belong to the CSI driver.
- `volumeHandle`: A string value that uniquely identifies the volume. This value
  must correspond to the value returned in the `volume.id` field of the
  `CreateVolumeResponse` by the CSI driver as defined in the [CSI spec](https://github.com/container-storage-interface/spec/blob/master/spec.md#createvolume).
  The value is passed as `volume_id` on all calls to the CSI volume driver when
  referencing the volume.
- `readOnly`: An optional boolean value indicating whether the volume is to be
  "ControllerPublished" (attached) as read only. Default is false. This value is
  passed to the CSI driver via the `readonly` field in the
  `ControllerPublishVolumeRequest`.
- `fsType`: If the PV's `VolumeMode` is `Filesystem` then this field may be used
  to specify the filesystem that should be used to mount the volume. If the
  volume has not been formatted and formatting is supported, this value will be
  used to format the volume.
  This value is passed to the CSI driver via the `VolumeCapability` field of
  `ControllerPublishVolumeRequest`, `NodeStageVolumeRequest`, and
  `NodePublishVolumeRequest`.
- `volumeAttributes`: A map of string to string that specifies static properties
  of a volume. This map must correspond to the map returned in the
  `volume.attributes` field of the `CreateVolumeResponse` by the CSI driver as
  defined in the [CSI spec](https://github.com/container-storage-interface/spec/blob/master/spec.md#createvolume).
  The map is passed to the CSI driver via the `volume_attributes` field in the
  `ControllerPublishVolumeRequest`, `NodeStageVolumeRequest`, and
  `NodePublishVolumeRequest`.
- `controllerPublishSecretRef`: A reference to the secret object containing
  sensitive information to pass to the CSI driver to complete the CSI
  `ControllerPublishVolume` and `ControllerUnpublishVolume` calls. This field is
  optional, and may be empty if no secret is required. If the secret object
  contains more than one secret, all secrets are passed.
- `nodeStageSecretRef`: A reference to the secret object containing
  sensitive information to pass to the CSI driver to complete the CSI
  `NodeStageVolume` call. This field is optional, and may be empty if no secret
  is required. If the secret object contains more than one secret, all secrets
  are passed.
- `nodePublishSecretRef`: A reference to the secret object containing
  sensitive information to pass to the CSI driver to complete the CSI
  `NodePublishVolume` call. This field is optional, and may be empty if no
  secret is required. If the secret object contains more than one secret, all
  secrets are passed.

#### CSI raw block volume support

{{< feature-state for_k8s_version="v1.14" state="beta" >}}

Starting with version 1.11, CSI introduced support for raw block volumes, which
relies on the raw block volume feature that was introduced in a previous version of
Kubernetes.  This feature will make it possible for vendors with external CSI drivers to
implement raw block volumes support in Kubernetes workloads.

CSI block volume support is feature-gated, but enabled by default. The two
feature gates which must be enabled for this feature are `BlockVolume` and
`CSIBlockVolume`.

Learn how to
[setup your PV/PVC with raw block volume support](/docs/concepts/storage/persistent-volumes/#raw-block-volume-support).

#### CSI ephemeral volumes

{{< feature-state for_k8s_version="v1.16" state="beta" >}}

This feature allows CSI volumes to be directly embedded in the Pod specification instead of a PersistentVolume. Volumes specified in this way are ephemeral and do not persist across Pod restarts.

Example:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: my-csi-app
spec:
  containers:
    - name: my-frontend
      image: busybox
      volumeMounts:
      - mountPath: "/data"
        name: my-csi-inline-vol
      command: [ "sleep", "1000000" ]
  volumes:
    - name: my-csi-inline-vol
      csi:
        driver: inline.storage.kubernetes.io
        volumeAttributes:
              foo: bar
```

This feature requires CSIInlineVolume feature gate to be enabled. It
is enabled by default starting with Kubernetes 1.16.

CSI ephemeral volumes are only supported by a subset of CSI drivers. Please see the list of CSI drivers [here](https://kubernetes-csi.github.io/docs/drivers.html).

# Developer resources
For more information on how to develop a CSI driver, refer to the [kubernetes-csi
documentation](https://kubernetes-csi.github.io/docs/)

#### Migrating to CSI drivers from in-tree plugins

{{< feature-state for_k8s_version="v1.14" state="alpha" >}}

The CSI Migration feature, when enabled, directs operations against existing in-tree
plugins to corresponding CSI plugins (which are expected to be installed and configured).
The feature implements the necessary translation logic and shims to re-route the
operations in a seamless fashion. As a result, operators do not have to make any
configuration changes to existing Storage Classes, PVs or PVCs (referring to
in-tree plugins) when transitioning to a CSI driver that supersedes an in-tree plugin.

In the alpha state, the operations and features that are supported include
provisioning/delete, attach/detach, mount/unmount and resizing of volumes.

In-tree plugins that support CSI Migration and have a corresponding CSI driver implemented
are listed in the "Types of Volumes" section above.

### FlexVolume {#flexVolume}

FlexVolume is an out-of-tree plugin interface that has existed in Kubernetes
since version 1.2 (before CSI). It uses an exec-based model to interface with
drivers. FlexVolume driver binaries must be installed in a pre-defined volume
plugin path on each node (and in some cases master).

Pods interact with FlexVolume drivers through the `flexvolume` in-tree plugin.
More details can be found [here](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-storage/flexvolume.md).

## Mount propagation

Mount propagation allows for sharing volumes mounted by a Container to
other Containers in the same Pod, or even to other Pods on the same node.

Mount propagation of a volume is controlled by `mountPropagation` field in Container.volumeMounts.
Its values are:

 * `None` - This volume mount will not receive any subsequent mounts
   that are mounted to this volume or any of its subdirectories by the host.
   In similar fashion, no mounts created by the Container will be visible on
   the host. This is the default mode.

   This mode is equal to `private` mount propagation as described in the
   [Linux kernel documentation](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)

 * `HostToContainer` - This volume mount will receive all subsequent mounts
   that are mounted to this volume or any of its subdirectories.

   In other words, if the host mounts anything inside the volume mount, the
   Container will see it mounted there.

   Similarly, if any Pod with `Bidirectional` mount propagation to the same
   volume mounts anything there, the Container with `HostToContainer` mount
   propagation will see it.

   This mode is equal to `rslave` mount propagation as described in the
   [Linux kernel documentation](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)

 * `Bidirectional` - This volume mount behaves the same the `HostToContainer` mount.
   In addition, all volume mounts created by the Container will be propagated
   back to the host and to all Containers of all Pods that use the same volume.

   A typical use case for this mode is a Pod with a FlexVolume or CSI driver or
   a Pod that needs to mount something on the host using a `hostPath` volume.

   This mode is equal to `rshared` mount propagation as described in the
   [Linux kernel documentation](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)

{{< caution >}}
`Bidirectional` mount propagation can be dangerous. It can damage
the host operating system and therefore it is allowed only in privileged
Containers. Familiarity with Linux kernel behavior is strongly recommended.
In addition, any volume mounts created by Containers in Pods must be destroyed
(unmounted) by the Containers on termination.
{{< /caution >}}

### Configuration
Before mount propagation can work properly on some deployments (CoreOS,
RedHat/Centos, Ubuntu) mount share must be configured correctly in
Docker as shown below.

Edit your Docker's `systemd` service file.  Set `MountFlags` as follows:
```shell
MountFlags=shared
```
Or, remove `MountFlags=slave` if present.  Then restart the Docker daemon:
```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```



{{% capture whatsnext %}}
* Follow an example of [deploying WordPress and MySQL with Persistent Volumes](/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/).
{{% /capture %}}
