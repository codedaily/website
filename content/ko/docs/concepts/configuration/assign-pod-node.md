---
title: 노드에 파드 할당하기
content_template: templates/concept
weight: 30
---


{{% capture overview %}}

You can constrain a {{< glossary_tooltip text="Pod" term_id="pod" >}} to only be able to run on particular {{< glossary_tooltip text="Node(s)" term_id="node" >}}, or to prefer to run on particular nodes.
There are several ways to do this, and the recommended approaches all use [label selectors](/docs/concepts/overview/working-with-objects/labels/) to make the selection.
스케줄러가 자동으로 적절한 배치를 수행하므로 일반적으로 이러한 제한은 필요하지 않다(예: 파드를 노드에 분산시키고, 여유 자원이 부족한 노드에 파드를 배치하지 않음). 그러나, 파드가 실행되는 노드에서 더 많은 제어를 원하는 경우가 있다. 예를 들어, 

but there are some circumstances where you may want more control on a node where a pod lands, e.g. to ensure that a pod ends up on a machine with an SSD attached to it, or to co-locate pods from two different services that communicate a lot into the same availability zone.

{{% /capture %}}

{{% capture body %}}

## nodeSelector

`nodeSelector` 는 가장 간단한 권장 노드 선택 제한 양식이다. `nodeSelector` 는 PodSpec의 필드이다. 키-값의 맵으로 지정한다. 파드가 노드에서 실행될 수 있으려면, 노드에 표시된 각 키-값 쌍이 레이블로 있어야 한다(추가 레이블도 있을 수 있음).
For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels (it can have
additional labels as well). 
가장 일반적인 사용법은 하나의 키-값으로 이루어진다.

`nodeSelector`를 사용하는 예를 살펴 보자.

### Step Zero: 전제조건

이 예는 Kubernetes 파드에 대한 기본 지식이 있고, [Kubernetes 클러스터를 설정](/ko/docs/setup/)했다고 가정한다.

### Step One: 노드에 레이블 부여

`kubectl get nodes`를 실행하여 클러스터의 노드 이름을  가져온다. `kubectl label nodes <node-name> <label-key>=<label-value>`를 실행하여 선택한 노드에 라벨을 추가한다. 예를 들어, 노드의 이름이 'kubernetes-foo-node-1.c.a-robinson.internal'이고 원하는 레이블이 'disktype=ssd'인 경우, `kubectl label nodes kubernetes-foo-node-1.c.a-robinson.internal disktype=ssd` 같이 실행할 수 있다.

`kubectl get nodes --show-labels`를 실행하여 노드에 레이블이 있는 확인할 수 있다. 또한, `kubectl describe node "nodename"`을 실행하여 노드의 전체 레이블 목록을 볼 수도 있다.

### Step Two: 파드 구성에 nodeSelector 필드 추가

실행할 파드 config 파일에 nodeSelector 섹션을 추가한다. 예를들어, 아래가 나의 파드 구성이라면:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
```

다음처럼 nodeSelector를 추가한다:

{{< codenew file="pods/pod-nginx.yaml" >}}

`kubectl apply -f https://k8s.io/examples/pods/pod-nginx.yaml` 실행될 때, 파드는 레이블을 부착한 노드에 예약된다. `kubectl get pods -o wide` 실행하고, 파드가 할당된 "NODE"를 보면 작동하는지 확인할 수 있다.

## Interlude: 내장 노드 레이블 {#built-in-node-labels}

[부착한](#step-one-attach-label-to-the-node) 레이블 외에도, 노드에는 표준 레이블 세트가 미리 채워져 있다. 이 레이블은 다음과 같다.

* [`kubernetes.io/hostname`](/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-hostname)
* [`failure-domain.beta.kubernetes.io/zone`](/docs/reference/kubernetes-api/labels-annotations-taints/#failure-domain-beta-kubernetes-io-zone)
* [`failure-domain.beta.kubernetes.io/region`](/docs/reference/kubernetes-api/labels-annotations-taints/#failure-domain-beta-kubernetes-io-region)
* [`beta.kubernetes.io/instance-type`](/docs/reference/kubernetes-api/labels-annotations-taints/#beta-kubernetes-io-instance-type)
* [`kubernetes.io/os`](/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-os)
* [`kubernetes.io/arch`](/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-arch)

{{< note >}}
이 라벨의 값은 클라우드 업체마다 다르고 정확하지 않을 수도 있다. 예를 들어, `kubernetes.io/hostname`의 값은 일부 환경에서 Node 이름과 동일하게 쓰일 수 있고 다른 환경에서는 또 다른 값으로 쓰일 수 있다.
{{< /note >}}

## Node 격리/제한

노드 객체에 레이블을 추가하면 특정 노드나 노드의 그룹에 파드를 타겟팅할 수 있다. 이를 통해 특정 파드가 일정한 격리, 보안 또는 규제 속성을 가진 노드에서만 실행되도록 할 수 있다. 이 목적으로 레이블을 사용하려는 경우, 노드에서 kubelet 프로세스로 수정할 수 없는 레이블 키를 선택하는 것을 강력 추천한다. 
This prevents a compromised node from using its kubelet credential to set those labels on its own Node object,
and influencing the scheduler to schedule workloads to the compromised node.

`NodeRestriction` 승인 플러그인은 kubelets이 `node-restriction.kubernetes.io/` 접두사를 사용하여 레이블을 설정하거나 수정하는 것을 막는다. 노드 격리에 해당 레이블을 사용하려면:

1. NodeRestriction을 사용할 수 있도록 Kubernetes v1.11 이상을 사용하고 있는지 확인한다.
2. [Node authorizer](/docs/reference/access-authn-authz/node/)를 사용하고 [NodeRestriction admission plugin](/docs/reference/access-authn-authz/admission-controllers/#noderestriction)을 _enabled_로 설정했는지 확인한다.
3. `node-restriction.kubernetes.io/` 접두사 아래에 레이블을 노드 오브젝트에 추가하고, 해당 레이블을 노드 셀렉터가 사용한다. 
  예를 들어, `example.com.node-restriction.kubernetes.io/fips=true`이거나 `example.com.node-restriction.kubernetes.io/pci-dss=true`.

## Affinity와 anti-affinity

`nodeSelector`는 특정 레이블이 있는 노드로 파드를 제한하는 매우 간단한 방법을 제공한다. affinity/anti-affinity 기능은, 표현할 수 있는 제약유형을 크게 확장한다. 주요 개선사항은 다음과 같다.

1. 표현식이 좀 더 다양하다("exact match의 AND"는 아니다).
2. you can indicate that the rule is "soft"/"preference" rather than a hard requirement, so if the scheduler can't satisfy it, the pod will still be scheduled
3. you can constrain against labels on other pods running on the node (or other topological domain), rather than against labels on the node itself, which allows rules about which pods can and cannot be co-located

The affinity feature consists of two types of affinity, "node affinity" and "inter-pod affinity/anti-affinity".
Node affinity is like the existing `nodeSelector` (but with the first two benefits listed above),
while inter-pod affinity/anti-affinity constrains against pod labels rather than node labels, as
described in the third item listed above, in addition to having the first and second properties listed above.

### Node affinity

Node affinity는 개념적으로 `nodeSelector`와 유사하다. -- 노드의 라벨을 기반으로 하여 it allows you to constrain which nodes your
pod is eligible to be scheduled on, based on labels on the node.

There are currently two types of node affinity, called `requiredDuringSchedulingIgnoredDuringExecution` and
`preferredDuringSchedulingIgnoredDuringExecution`. You can think of them as "hard" and "soft" respectively,
in the sense that the former specifies rules that *must* be met for a pod to be scheduled onto a node (just like
`nodeSelector` but using a more expressive syntax), while the latter specifies *preferences* that the scheduler
will try to enforce but will not guarantee. The "IgnoredDuringExecution" part of the names means that, similar
to how `nodeSelector` works, if labels on a node change at runtime such that the affinity rules on a pod are no longer
met, the pod will still continue to run on the node. In the future we plan to offer
`requiredDuringSchedulingRequiredDuringExecution` which will be just like `requiredDuringSchedulingIgnoredDuringExecution`
except that it will evict pods from nodes that cease to satisfy the pods' node affinity requirements.

Thus an example of `requiredDuringSchedulingIgnoredDuringExecution` would be "only run the pod on nodes with Intel CPUs"
and an example `preferredDuringSchedulingIgnoredDuringExecution` would be "try to run this set of pods in failure
zone XYZ, but if it's not possible, then allow some to run elsewhere".

Node affinity is specified as field `nodeAffinity` of field `affinity` in the PodSpec.

다음은 node affinity를 사용하는 파드의 예이다:

{{< codenew file="pods/pod-with-node-affinity.yaml" >}}

This node affinity rule says the pod can only be placed on a node with a label whose key is
`kubernetes.io/e2e-az-name` and whose value is either `e2e-az1` or `e2e-az2`. In addition,
among nodes that meet that criteria, nodes with a label whose key is `another-node-label-key` and whose
value is `another-node-label-value` should be preferred.

You can see the operator `In` being used in the example. The new node affinity syntax supports the following operators: `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`.
You can use `NotIn` and `DoesNotExist` to achieve node anti-affinity behavior, or use
[node taints](/docs/concepts/configuration/taint-and-toleration/) to repel pods from specific nodes.

If you specify both `nodeSelector` and `nodeAffinity`, *both* must be satisfied for the pod
to be scheduled onto a candidate node.

If you specify multiple `nodeSelectorTerms` associated with `nodeAffinity` types, then the pod can be scheduled onto a node **if one of** the `nodeSelectorTerms` is satisfied.

If you specify multiple `matchExpressions` associated with `nodeSelectorTerms`, then the pod can be scheduled onto a node **only if all** `matchExpressions` can be satisfied.

If you remove or change the label of the node where the pod is scheduled, the pod won't be removed. In other words, the affinity selection works only at the time of scheduling the pod.

The `weight` field in `preferredDuringSchedulingIgnoredDuringExecution` is in the range 1-100. For each node that meets all of the scheduling requirements (resource request, RequiredDuringScheduling affinity expressions, etc.), the scheduler will compute a sum by iterating through the elements of this field and adding "weight" to the sum if the node matches the corresponding MatchExpressions. This score is then combined with the scores of other priority functions for the node. The node(s) with the highest total score are the most preferred.

### Inter-pod affinity and anti-affinity

Inter-pod affinity and anti-affinity allow you to constrain which nodes your pod is eligible to be scheduled *based on
labels on pods that are already running on the node* rather than based on labels on nodes. The rules are of the form
"this pod should (or, in the case of anti-affinity, should not) run in an X if that X is already running one or more pods that meet rule Y".
Y is expressed as a LabelSelector with an optional associated list of namespaces; unlike nodes, because pods are namespaced
(and therefore the labels on pods are implicitly namespaced),
a label selector over pod labels must specify which namespaces the selector should apply to. Conceptually X is a topology domain
like node, rack, cloud provider zone, cloud provider region, etc. You express it using a `topologyKey` which is the
key for the node label that the system uses to denote such a topology domain, e.g. see the label keys listed above
in the section [Interlude: built-in node labels](#built-in-node-labels).

{{< note >}}
Inter-pod affinity and anti-affinity require substantial amount of
processing which can slow down scheduling in large clusters significantly. We do
not recommend using them in clusters larger than several hundred nodes.
{{< /note >}}

{{< note >}}
Pod anti-affinity requires nodes to be consistently labelled, i.e. every node in the cluster must have an appropriate label matching `topologyKey`. If some or all nodes are missing the specified `topologyKey` label, it can lead to unintended behavior.
{{< /note >}}

As with node affinity, there are currently two types of pod affinity and anti-affinity, called `requiredDuringSchedulingIgnoredDuringExecution` and
`preferredDuringSchedulingIgnoredDuringExecution` which denote "hard" vs. "soft" requirements.
See the description in the node affinity section earlier.
An example of `requiredDuringSchedulingIgnoredDuringExecution` affinity would be "co-locate the pods of service A and service B
in the same zone, since they communicate a lot with each other"
and an example `preferredDuringSchedulingIgnoredDuringExecution` anti-affinity would be "spread the pods from this service across zones"
(a hard requirement wouldn't make sense, since you probably have more pods than zones).

Inter-pod affinity is specified as field `podAffinity` of field `affinity` in the PodSpec.
And inter-pod anti-affinity is specified as field `podAntiAffinity` of field `affinity` in the PodSpec.

#### An example of a pod that uses pod affinity:

{{< codenew file="pods/pod-with-pod-affinity.yaml" >}}

The affinity on this pod defines one pod affinity rule and one pod anti-affinity rule. In this example, the
`podAffinity` is `requiredDuringSchedulingIgnoredDuringExecution`
while the `podAntiAffinity` is `preferredDuringSchedulingIgnoredDuringExecution`. The
pod affinity rule says that the pod can be scheduled onto a node only if that node is in the same zone
as at least one already-running pod that has a label with key "security" and value "S1". (More precisely, the pod is eligible to run
on node N if node N has a label with key `failure-domain.beta.kubernetes.io/zone` and some value V
such that there is at least one node in the cluster with key `failure-domain.beta.kubernetes.io/zone` and
value V that is running a pod that has a label with key "security" and value "S1".) The pod anti-affinity
rule says that the pod prefers not to be scheduled onto a node if that node is already running a pod with label
having key "security" and value "S2". (If the `topologyKey` were `failure-domain.beta.kubernetes.io/zone` then
it would mean that the pod cannot be scheduled onto a node if that node is in the same zone as a pod with
label having key "security" and value "S2".) See the
[design doc](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md)
for many more examples of pod affinity and anti-affinity, both the `requiredDuringSchedulingIgnoredDuringExecution`
flavor and the `preferredDuringSchedulingIgnoredDuringExecution` flavor.

The legal operators for pod affinity and anti-affinity are `In`, `NotIn`, `Exists`, `DoesNotExist`.

In principle, the `topologyKey` can be any legal label-key. However,
for performance and security reasons, there are some constraints on topologyKey:

1. For affinity and for `requiredDuringSchedulingIgnoredDuringExecution` pod anti-affinity,
empty `topologyKey` is not allowed.
2. For `requiredDuringSchedulingIgnoredDuringExecution` pod anti-affinity, the admission controller `LimitPodHardAntiAffinityTopology` was introduced to limit `topologyKey` to `kubernetes.io/hostname`. If you want to make it available for custom topologies, you may modify the admission controller, or simply disable it.
3. For `preferredDuringSchedulingIgnoredDuringExecution` pod anti-affinity, empty `topologyKey` is interpreted as "all topologies" ("all topologies" here is now limited to the combination of `kubernetes.io/hostname`, `failure-domain.beta.kubernetes.io/zone` and `failure-domain.beta.kubernetes.io/region`).
4. Except for the above cases, the `topologyKey` can be any legal label-key.

In addition to `labelSelector` and `topologyKey`, you can optionally specify a list `namespaces`
of namespaces which the `labelSelector` should match against (this goes at the same level of the definition as `labelSelector` and `topologyKey`).
If omitted or empty, it defaults to the namespace of the pod where the affinity/anti-affinity definition appears.

All `matchExpressions` associated with `requiredDuringSchedulingIgnoredDuringExecution` affinity and anti-affinity
must be satisfied for the pod to be scheduled onto a node.

#### More Practical Use-cases

Interpod Affinity and AntiAffinity can be even more useful when they are used with higher
level collections such as ReplicaSets, StatefulSets, Deployments, etc.  One can easily configure that a set of workloads should
be co-located in the same defined topology, eg., the same node.

##### Always co-located in the same node

In a three node cluster, a web application has in-memory cache such as redis. We want the web-servers to be co-located with the cache as much as possible.

Here is the yaml snippet of a simple redis deployment with three replicas and selector label `app=store`. The deployment has `PodAntiAffinity` configured to ensure the scheduler does not co-locate replicas on a single node.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  selector:
    matchLabels:
      app: store
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis:3.2-alpine
```

The below yaml snippet of the webserver deployment has `podAntiAffinity` and `podAffinity` configured. This informs the scheduler that all its replicas are to be co-located with pods that have selector label `app=store`. This will also ensure that each web-server replica does not co-locate on a single node.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web-store
  replicas: 3
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-store
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: web-app
        image: nginx:1.12-alpine
```

If we create the above two deployments, our three node cluster should look like below.

|       node-1         |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| *webserver-1*        |   *webserver-2*     |    *webserver-3*   |
|  *cache-1*           |     *cache-2*       |     *cache-3*      |

As you can see, all the 3 replicas of the `web-server` are automatically co-located with the cache as expected.

```
kubectl get pods -o wide
```
The output is similar to this:
```
NAME                           READY     STATUS    RESTARTS   AGE       IP           NODE
redis-cache-1450370735-6dzlj   1/1       Running   0          8m        10.192.4.2   kube-node-3
redis-cache-1450370735-j2j96   1/1       Running   0          8m        10.192.2.2   kube-node-1
redis-cache-1450370735-z73mh   1/1       Running   0          8m        10.192.3.1   kube-node-2
web-server-1287567482-5d4dz    1/1       Running   0          7m        10.192.2.3   kube-node-1
web-server-1287567482-6f7v5    1/1       Running   0          7m        10.192.4.3   kube-node-3
web-server-1287567482-s330j    1/1       Running   0          7m        10.192.3.2   kube-node-2
```

##### Never co-located in the same node

The above example uses `PodAntiAffinity` rule with `topologyKey: "kubernetes.io/hostname"` to deploy the redis cluster so that
no two instances are located on the same host.
See [ZooKeeper tutorial](/docs/tutorials/stateful-application/zookeeper/#tolerating-node-failure)
for an example of a StatefulSet configured with anti-affinity for high availability, using the same technique.

## nodeName

`nodeName` is the simplest form of node selection constraint, but due
to its limitations it is typically not used.  `nodeName` is a field of
PodSpec.  If it is non-empty, the scheduler ignores the pod and the
kubelet running on the named node tries to run the pod.  Thus, if
`nodeName` is provided in the PodSpec, it takes precedence over the
above methods for node selection.

Some of the limitations of using `nodeName` to select nodes are:

-   If the named node does not exist, the pod will not be run, and in
    some cases may be automatically deleted.
-   If the named node does not have the resources to accommodate the
    pod, the pod will fail and its reason will indicate why,
    e.g. OutOfmemory or OutOfcpu.
-   Node names in cloud environments are not always predictable or
    stable.

Here is an example of a pod config file using the `nodeName` field:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: kube-01
```

The above pod will run on the node kube-01.

{{% /capture %}}

{{% capture whatsnext %}}

[Taints](/docs/concepts/configuration/taint-and-toleration/) allow a Node to *repel* a set of Pods.

The design documents for
[node affinity](https://git.k8s.io/community/contributors/design-proposals/scheduling/nodeaffinity.md)
and for [inter-pod affinity/anti-affinity](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md) contain extra background information about these features.

Once a Pod is assigned to a Node, the kubelet runs the Pod and allocates node-local resources.
The [topology manager](/docs/tasks/administer-cluster/topology-manager/) can take part in node-level
resource allocation decisions. 

{{% /capture %}}
