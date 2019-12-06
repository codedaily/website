---
title: kubectl 개요
content_template: templates/concept
weight: 20
card:
  name: reference
  weight: 20
---

{{% capture overview %}}
Kubectl은 Kubernetes 클러스터에 명령을 실행하기 위한 커맨드라인 인터페이스이다. `kubectl`은 $HOME/.kube 디렉토리에서 config라는 파일을 찾는다. KUBECONFIG 환경 변수를 설정하거나 [`--kubeconfig`](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) 플래그를 설정하여 다른 [kubeconfig](/ko/docs/concepts/configuration/organize-cluster-access-kubeconfig/) 파일을 지정할 수 있다.

이 개요는 `kubectl` 구문을 다루고 명령 작업을 설명하며 일반적인 예를 제공한다. 지원되는 모든 플래그와 하위 명령에 대한 자세한 내용은 [kubectl](/ko/docs/reference/generated/kubectl/kubectl-commands/) 레퍼런스 문서를 참고하라. 설치는 [kubectl 설치](/ko/docs/tasks/kubectl/install/)를 참고하라.

{{% /capture %}}

{{% capture body %}}

## 구문

터미널 창에서 `kubectl` 명령을 실행하려면 다음 구문을 사용하라:

```shell
kubectl [command] [TYPE] [NAME] [flags]
```

`command`, `TYPE`, `NAME`, `flags` 의 의미는 다음과 같다:

* `command`: 하나 이상의 리소스에서 수행하려는 작업을 지정한다. 예를 들어, `create`, `get`, `describe`, `delete` 이다.

* `TYPE`: [리소스 타입](#리소스-타입)을 지정한다. 리소스 타입은 대소문자를 구분하지 않으며, 단수형, 복수형 또는 약식 양식을 지정할 수 있다. 예를들어, 다음 명령은 동일한 출력을 생성한다:

      ```shell
      kubectl get pod pod1
      kubectl get pods pod1
      kubectl get po pod1
      ```

* `NAME`: 리소스의 이름을 지정한다. 이름은 대소문자를 구분한다. 이름을 생략하면, 모든 리소스의 상세정보가 표시된다. 예를들어, `kubectl get pods`.

   다중 리소스 제어를 수행할 때, 타입과 이름으로 각 리소스를 지정하거나 하나 이상의 파일을 지정할 수 있다:

   * 타입과 이름으로 리소스를 지정할 때:

      * 리소스가 모두 같은 유형을 그룹화할 경우: `TYPE1 name1 name2 name<#>`.<br/>
      Example: `kubectl get pod example-pod1 example-pod2`

      * 다중 리소스 타입을 개별적으로 지정할 경우:  `TYPE1/name1 TYPE1/name2 TYPE2/name3 TYPE<#>/name<#>`.<br/>
      Example: `kubectl get pod/example-pod1 replicationcontroller/example-rc1`

   * 하나 이상의 파일로 리소스를 지정할 때: `-f file1 -f file2 -f file<#>`

      * YAML 구문은 사용자 친화적이기 때문에 구성파일은 [JSON 보다 YAML을 사용하라](/docs/concepts/configuration/overview/#general-configuration-tips).<br/>
     Example: `kubectl get pod -f ./pod.yaml`

* `flags`: 옵션 플래그를 지정한다. 예를들어, `-s` 또는 `--server` 플래그를 사용하여 Kubernetes API 서버의 주소와 포트를 지정할 수 있다.<br/>

{{< caution >}}
커맨드라인에서 지정하는 플래그는 기본값 및 해당 환경 변수를 대체한다.
{{< /caution >}}

도움이 필요하면 터미널 창에서 `kubectl help`을 실행한다.

## 명령어

다음 표는 모든 `kubectl` 작업에 대한 간단한 설명과 일반적인 구문이 포함되어 있다:

명령어       | 구문    |       설명
-------------------- | -------------------- | --------------------
`annotate`    | `kubectl annotate (-f FILENAME \| TYPE NAME \| TYPE/NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--overwrite] [--all] [--resource-version=version] [flags]` | 하나 이상의 리소스의 어노테이션을 추가하거나 업데이트 한다.
`api-versions`    | `kubectl api-versions [flags]` | 사용가능한 API 버전을 리스트로 나열한다.
`apply`            | `kubectl apply -f FILENAME [flags]`| 파일 또는 stdin에서 리소스에 대한 구성 변경 사항을 적용한다.
`attach`        | `kubectl attach POD -c CONTAINER [-i] [-t] [flags]` | 실행 중인 컨테이너에 연결하여 출력 스트림을 보거나 컨테이너(stdin)와 상호 작용한다.
`autoscale`    | `kubectl autoscale (-f FILENAME \| TYPE NAME \| TYPE/NAME) [--min=MINPODS] --max=MAXPODS [--cpu-percent=CPU] [flags]` | 복제 컨트롤러에서 관리하는 파드 세트를 자동으로 조정한다.
`cluster-info`    | `kubectl cluster-info [flags]` | 클러스터의 마스터와 서비스에 대한 엔드포인트 정보를 표시한다.
`config`        | `kubectl config SUBCOMMAND [flags]` | kubeconfig 파일을 수정한다. 상세정보는 개별 하위명령을 참조한다.
`create`        | `kubectl create -f FILENAME [flags]` | 파일이나 stdin에서 하나 이상의 리소스를 생성한다.
`delete`        | `kubectl delete (-f FILENAME \| TYPE [NAME \| /NAME \| -l label \| --all]) [flags]` | 파일, stdin 또는 라벨 선택기, 이름, 리소스 선택기, 또는 리소스를 지정하여 리소스를 삭제한다.
`describe`    | `kubectl describe (-f FILENAME \| TYPE [NAME_PREFIX \| /NAME \| -l label]) [flags]` | 하나 이상의 리소스의 자세한 상태를 표시한다.
`diff`        | `kubectl diff -f FILENAME [flags]`| 라이브 구성에 대한 파일 또는 STDIN으로 비교한다(**BETA**)
`edit`        | `kubectl edit (-f FILENAME \| TYPE NAME \| TYPE/NAME) [flags]` | 기본편집기를 사용하여 서버에서 하나 이상의 리소스의 정의를 편집하고 업데이트 한다.
`exec`        | `kubectl exec POD [-c CONTAINER] [-i] [-t] [flags] [-- COMMAND [args...]]` | 파드의 컨테이너에 대해 명령을 실행한다.
`explain`    | `kubectl explain  [--recursive=false] [flags]` | 다양한 리소스의 문서를 얻는다. 파드, 노드, 서비스, 기타 등등.
`expose`        | `kubectl expose (-f FILENAME \| TYPE NAME \| TYPE/NAME) [--port=port] [--protocol=TCP\|UDP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type] [flags]` | 복제 컨트롤러, 서비스, 또는 파드를 새로운 Kubernetes 서비스로 익스포트한다.
`get`        | `kubectl get (-f FILENAME \| TYPE [NAME \| /NAME \| -l label]) [--watch] [--sort-by=FIELD] [[-o \| --output]=OUTPUT_FORMAT] [flags]` | 하나 이상의 리소스를 리스트로 나열한다.
`label`        | `kubectl label (-f FILENAME \| TYPE NAME \| TYPE/NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--overwrite] [--all] [--resource-version=version] [flags]` | 하나 이상의 리소스의 라벨을 추가하거나 업데이트 한다.
`logs`        | `kubectl logs POD [-c CONTAINER] [--follow] [flags]` | 파드에 있는 컨테이너의 로그를 출력한다.
`patch`        | `kubectl patch (-f FILENAME \| TYPE NAME \| TYPE/NAME) --patch PATCH [flags]` | 전략적 머지 패치 프로세스를 사용하여 리소스의 하나 이상의 필드를 업데이트 한다.
`port-forward`    | `kubectl port-forward POD [LOCAL_PORT:]REMOTE_PORT [...[LOCAL_PORT_N:]REMOTE_PORT_N] [flags]` | 하나이상의 로컬 포트를 파드로 전달한다.
`proxy`        | `kubectl proxy [--port=PORT] [--www=static-dir] [--www-prefix=prefix] [--api-prefix=prefix] [flags]` | Kubernetes API 서버에 프록시를 실행한다.
`replace`        | `kubectl replace -f FILENAME` | 파일 또는 stdin에서 리소스를 바꾼다.
`rolling-update`    | `kubectl rolling-update OLD_CONTROLLER_NAME ([NEW_CONTROLLER_NAME] --image=NEW_CONTAINER_IMAGE \| -f NEW_CONTROLLER_SPEC) [flags]` | 지정된 복제 컨트롤러와 해당 파드를 점차적으로 교체하여 롤링 업데이트를 수행한다.
`run`        | `kubectl run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [flags]` | 클러스터에서 지정된 이미지를 실행한다.
`scale`        | `kubectl scale (-f FILENAME \| TYPE NAME \| TYPE/NAME) --replicas=COUNT [--resource-version=version] [--current-replicas=count] [flags]` | 지정된 복제 컨트롤러의 크기를 업데이트 한다.
`stop`        | `kubectl stop` | 더이상 사용하지 않음: 대신, `kubectl delete` 참고하라.
`version`        | `kubectl version [--client] [flags]` | 클라이언트와 서버에서 실행 중인 Kubenetes 버전을 표시한다.

알아두기: 명령 실행에 대한 자세한 내용은, [kubectl](/ko/docs/user-guide/kubectl/) 레퍼런스 문서를 참고하라.

## 리소스 타입

다음 표에는 지원되는 모든 리소스 타입과 해당 약어가 나열되어 있다.

(이 출력은 `kubectl api-resources`에서 검색 할 수 있으며 Kubernetes 1.13.3부터 정확하다.)

| 리소스 이름 | 약어 | API 그룹 | 네임스페이스 여부 | 리소스 종류 |
|---|---|---|---|---|
| `componentstatuses` | `cs` | | false | ComponentStatus |
| `configmaps` | `cm` | | true | ConfigMap |
| `endpoints` | `ep` | | true | Endpoints |
| `limitranges` | `limits` | | true | LimitRange |
| `namespaces` | `ns` | | false | Namespace |
| `nodes` | `no` | | false | Node |
| `persistentvolumeclaims` | `pvc` | | true | PersistentVolumeClaim |
| `persistentvolumes` | `pv` | | false | PersistentVolume |
| `pods` | `po` | | true | Pod |
| `podtemplates` | | | true | PodTemplate |
| `replicationcontrollers` | `rc` | | true| ReplicationController |
| `resourcequotas` | `quota` | | true | ResourceQuota |
| `secrets` | | | true | Secret |
| `serviceaccounts` | `sa` | | true | ServiceAccount |
| `services` | `svc` | | true | Service |
| `mutatingwebhookconfigurations` | | admissionregistration.k8s.io | false | MutatingWebhookConfiguration |
| `validatingwebhookconfigurations` | | admissionregistration.k8s.io | false | ValidatingWebhookConfiguration |
| `customresourcedefinitions` | `crd`, `crds` | apiextensions.k8s.io | false |  CustomResourceDefinition |
| `apiservices` | | apiregistration.k8s.io | false | APIService |
| `controllerrevisions` | | apps | true | ControllerRevision |
| `daemonsets` | `ds` | apps | true | DaemonSet |
| `deployments` | `deploy` | apps | true | Deployment |
| `replicasets` | `rs` | apps | true | ReplicaSet |
| `statefulsets` | `sts` | apps | true | StatefulSet |
| `tokenreviews` | | authentication.k8s.io | false | TokenReview |
| `localsubjectaccessreviews` | | authorization.k8s.io | true | LocalSubjectAccessReview |
| `selfsubjectaccessreviews` | | authorization.k8s.io | false | SelfSubjectAccessReview |
| `selfsubjectrulesreviews` | | authorization.k8s.io | false | SelfSubjectRulesReview |
| `subjectaccessreviews` | | authorization.k8s.io | false | SubjectAccessReview |
| `horizontalpodautoscalers` | `hpa` | autoscaling | true | HorizontalPodAutoscaler |
| `cronjobs` | `cj` | batch | true | CronJob |
| `jobs` | | batch | true | Job |
| `certificatesigningrequests` | `csr` | certificates.k8s.io | false | CertificateSigningRequest |
| `leases` | | coordination.k8s.io | true | Lease |
| `events` | `ev` | events.k8s.io | true | Event |
| `ingresses` | `ing` | extensions | true | Ingress |
| `networkpolicies` | `netpol` | networking.k8s.io | true | NetworkPolicy |
| `poddisruptionbudgets` | `pdb` | policy | true | PodDisruptionBudget |
| `podsecuritypolicies` | `psp` | policy | false | PodSecurityPolicy |
| `clusterrolebindings` | | rbac.authorization.k8s.io | false | ClusterRoleBinding |
| `clusterroles` | | rbac.authorization.k8s.io | false | ClusterRole |
| `rolebindings` | | rbac.authorization.k8s.io | true | RoleBinding |
| `roles` | | rbac.authorization.k8s.io | true | Role |
| `priorityclasses` | `pc` | scheduling.k8s.io | false | PriorityClass |
| `storageclasses` | `sc` | storage.k8s.io |  false | StorageClass |
| `volumeattachments` | | storage.k8s.io | false | VolumeAttachment |

## 출력 옵션

특정 명령의 출력을 형식화하거나 정렬하는 방법에 대해서는 다음 섹션을 사용하라. 다양한 출력 옵션을 지원하는 명령에 대한 자세한 내용은 [kubectl](/ko/docs/user-guide/kubectl/) 레퍼런스 문서를 참조하라.

### 포맷팅 출력

모든 `kubectl` 명령의 기본 출력 형식은 사람이 읽을 수 있는 일반 텍스트 형식이다. 특정 형식으로 터미널 창에 세부 사항을 출력하려면 지원되는 kubectl 명령에 `-o` 또는 `--output` 플래그를 추가할 수 있다.

#### 구문

```shell
kubectl [command] [TYPE] [NAME] -o <output_format>
```

`kubectl` 작업에 따라, 다음과 같은 출력 형식이 지원된다:

출력 포맷 | 설명
--------------| -----------
`-o custom-columns=<spec>` | 콤마로 구분된 [사용자정의 컬럼](#사용자정의-컬럼) 목록을 사용하여 테이블을 출력한다.
`-o custom-columns-file=<filename>` | `<filename>` 파일에서 [사용자정의 컬럼](#사용자정의-컬럼) 템플릿을 사용하여 테이블을 출력한다.
`-o json`     | JSON 형식의 API 객체를 출력한다.
`-o jsonpath=<template>` | [jsonpath](/ko/docs/reference/kubectl/jsonpath/) 표현식에 정의된 필드를 출력한다.
`-o jsonpath-file=<filename>` | `<filename>`파일에서 [jsonpath](/ko/docs/reference/kubectl/jsonpath/) 표현식에 정의된 필드를 출력한다.
`-o name`     | 리소스 이름만 출력한다.
`-o wide`     | 추가 정보가 포함된 일반 텍스트형식으로 출력된다. 파드의 경우, 노드의 이름이 포함된다.
`-o yaml`     | YAML 형식의 API 객체를 출력한다.

##### 예제

이 예제에서, 다음 명령은 단일 파드에 대한 세부정보를 YAML 형식 객체로 출력한다:

```shell
kubectl get pod web-pod-13je7 -o yaml
```

알아두기: 각 명령에서 지원되는 출력 형식에 대한 자세한 내용은 [kubectl](/ko/docs/user-guide/kubectl/) 레퍼런스 문서를 참조한다. 

#### 사용자정의 컬럼

사용자정의 컬럼을 정의하고 원하는 세부정보만 테이블에 출력하기 위해 `custom-columns` 옵션을 사용할 수 있다. 사용자정의 컬럼을 인라인으로 정의하거나 템플릿 파일을 사용하도록 선택할 수 있다: `-o custom-columns=<spec>` 이거나 `-o custom-columns-file=<filename>`.

##### 예제

인라인:

```shell
kubectl get pods <pod-name> -o custom-columns=NAME:.metadata.name,RSRC:.metadata.resourceVersion
```

템플릿 파일:

```shell
kubectl get pods <pod-name> -o custom-columns-file=template.txt
```

`template.txt` 파일에는 아래의 내용이 포함되어 있다:

```
NAME          RSRC
metadata.name metadata.resourceVersion
```

두 명령 중 하나를 실행한 결과는 다음과 같다:


```shell
NAME           RSRC
submit-queue   610995
```

#### 서버사이드 컬럼

`kubectl`은 서버의 객체에 대한 특정 컬럼 정보 수신을 지원한다. 이는 클라이언트가 출력할 수 있도록 주어진 리소스에 대해 서버가 해당 리소스와 관련된 컬럼과 행을 반환한다는 것을 의미한다. 이는 서버가 출력의 세부사항을 캡슐화하도록 하여, 동일한 클러스터를 사용한 교차 클라이언트에서 사람이 읽을 수 있는 출력을 일관되게 허용한다.
This means that for any given resource, the server will return columns and rows relevant to that resource, for the client to print.
This allows for consistent human-readable output across clients used against the same cluster, by having the server encapsulate the details of printing.

이 기능은 기본적으로 `kubectl` 1.11 버전이상에서 활성화되어 있다. 비활성화하려면, `--server-print=false` 플래그를 `kubectl get` 명령에 추가한다.

##### 예제

파드의 상태에 대한 정보를 출력하려면, 다음과 같은 명령을 사용한다:

```shell
kubectl get pods <pod-name> --server-print=false
```

출력은 다음과 같다:

```shell
NAME       READY     STATUS              RESTARTS   AGE
pod-name   1/1       Running             0          1m
```

### 리스트 객체 정렬

터미널 창에서 정렬된 리스트로 객체를 출력하려면, `--sort-by` 플래그를 지원되는 `kubectl` 명령에 추가한다. `--sort-by` 플래그와 함께 숫자 또는 문자열 필드를 지정하여 객체를 정렬하라. 필드를 지정하려면, [jsonpath](/ko/docs/reference/kubectl/jsonpath/) 표현식을 사용한다.

#### 구문

```shell
kubectl [command] [TYPE] [NAME] --sort-by=<jsonpath_exp>
```

##### 예제

파드의 리스트를 이름으로 정렬하여 출력하길 원한다면, 다음과 같이 실행하라:

```shell
kubectl get pods --sort-by=.metadata.name
```

## 예제: Common operations

일반적으로 사용되는 `kubectl` 조작 실행에 익숙해지길 원한다면, 다음 예제 세트에 익숙해져라:

`kubectl apply` - 파일 또는 stdin 에서 리소스를 적용하거나 업데이트한다.

```shell
# example-service.yaml을 이용하여 서비스를 생성한다.
kubectl apply -f example-service.yaml

# example-controller.yaml을 이용하여 복제 컨트롤러를 생성한다.
kubectl apply -f example-controller.yaml

# <directory> 폴더안에 있는 .yaml, .yml 또는 .json 파일에 정의된 객체를 생성한다.
kubectl apply -f <directory>
```

`kubectl get` - 하나이상의 리소스 리스트를 출력한다.

```shell
# 일반 텍스트 형식의 모든 파드 목록을 출력한다.
kubectl get pods

# 일반 텍스트 형식의 모든 파드 목록을 출력하고 추가 정보를 포함한다(노드명처럼).
kubectl get pods -o wide

# 지정된 이름의 복제 컨트롤러 리스트를 일반 텍스트형식으로 출력한다. 팁: 'replicationcontroller'를 'rc' 알리아스로 짧게 사용할 수 있다.
kubectl get replicationcontroller <rc-name>

# 복제 컨트롤러와 서비스를 함께 일반 텍스트 형식의 리스트로 출력한다.
kubectl get rc,services

# 초기화되지 않은 것을 포함한 모든 데몬 셋을 일반 텍스트 형식으로 출력한다.
kubectl get ds --include-uninitialized

# server01 노드에 실행중인 모든 파드를 출력한다.
kubectl get pods --field-selector=spec.nodeName=server01
```

`kubectl describe` - 초기화되지 않은 리소스를 포함하여 하나 이상의 리소스에 대한 자세한 상태를 기본적으로 표시한다.

```shell
# <node-name>의 이름으로 된 노드의 상세정보를 표시한다.
kubectl describe nodes <node-name>

# <pod-name>의 이름으로 딘 파드의 상세정보를 표시한다.
kubectl describe pods/<pod-name>

# <rc-name>이라는 복제 컨트롤러에서 관리하는 모든 파드의 세부정보를 표시한다. 
# 알아두기: 복제 컨트롤러로 생성된 모든 파드에는 복제 컨트롤러의 이름이 접두사로 붙는다.
kubectl describe pods <rc-name>

# 초기화되지 않은 파드를 제외한 모든 포드를 설명한다.
kubectl describe pods --include-uninitialized=false
```

{{< note >}}
`kubectl get` 명령은 일반적으로 동일한 리소스 타입의 자원에서 하나 이상의 자원을 검색하는 데 사용된다. 예를 들어, `-o` 또는 `--output` 플래그를 사용하여 출력 형식을 사용자정의 할 수 있는 다양한 플래그 셋이 있다. `-w` 또는 `--watch` 플래그를 지정하여 특정 객체에 대한 업데이트를 볼 수 있다. `kubectl describe` 명령은 지정된 자원의 많은 관련된 부분을 설명하는데  중점을 둔다. API 서버에 대한 여러 API를 호출하여 사용자에 대한 보기를 빌드 할 수 있습니다. 예를 들어, `kubectl describe node` 명령은 노드에 대한 정보뿐만 아니라, 노드에서 실행 중인 파드, 노드에 대해 생성된 이벤트 등의 정보도 검색한다.
{{< /note >}}

`kubectl delete` - 파일, stdin에서 또는 라벨 선택기, 이름, 자원 선택기 또는 자원을 지정하여 자원을 삭제한다.

```shell
# pod.yaml 파일에 지정된 타입과 이름을 사용하여 파드를 삭제한다.
kubectl delete -f pod.yaml

# 라벨 이름이 <label-name> 인 모든 파드 및 서비스를 삭제한다.
kubectl delete pods,services -l name=<label-name>

# 초기화되지 않은 항목을 포함하여 라벨 이름이 <label-name> 인 모든 파드 및 서비스를 삭제한다.
kubectl delete pods,services -l name=<label-name> --include-uninitialized

# 초기화되지 않은 항목을 포함하여 모든 파드를 삭제한다.
kubectl delete pods --all
```

`kubectl exec` - 파드에 있는 컨테이너에 대하여 명령을 실행한다.

```shell
# <pod-name> 이름으로 된 파드에서 'date'를 실행하여 출력을 가져온다. 기본적으로, 출력은 첫번째 컨테이너에서 출력된다.
kubectl exec <pod-name> date

# <pod-name> 이름으로 된 파드의 <container-name> 이름으로 된 컨테이너에서 'date' 를 실행하여 출력을 가져온다.
kubectl exec <pod-name> -c <container-name> date

# 대화식 TTY를 가져와 <pod-name> 이름으로 된 파드에서 /bin/bash를 실행한다. 기본적으로, 출력은 첫번째 컨테이너에서 출력된다.
kubectl exec -ti <pod-name> /bin/bash
```

`kubectl logs` - 파드에 있는 컨테이너의 로그를 출력한다.

```shell
# <pod-name> 이름으로 된 파드의 로그를 스냅샷하여 출력한다.
kubectl logs <pod-name>

# <pod-name> 이름으로 된 파드의 로그의 스트리밍을 시작한다. 이것은 리눅스 명령어의 'tail -f'와 유사하다.
kubectl logs -f <pod-name>
```

## 예제: 생성과 플러그인 사용

`kubectl` 플러그인 작성과 사용에 익숙해지려면 다음 예제 세트를 사용하라:

```shell
# 모든 언어로 간단한 플러그인을 만들고 결과 실행 파일의 이름을 지정한다.
# 접두사 "kubectl-"으로 시작하도록 한다.
cat ./kubectl-hello
#!/bin/bash

# 이 플러그인은 "hello world"라는 단어를 인쇄한다.
echo "hello world"

# 플러그인을 작성하고, 실행 가능하도록 만든다.
sudo chmod +x ./kubectl-hello

# 그리고 PATH의 위치로 옮긴다.
sudo mv ./kubectl-hello /usr/local/bin

# 우리는 이제 kubectl 플러그인을 만들고 "설치"했습니다. 
# kubectl에서 플러그인을 일반 명령어처럼 호출하여 플러그인 사용할 수 있다.
kubectl hello
```
```
hello world
```

```
# PATH에서 플러그인을 간단히 제거할 수 있다.
sudo rm /usr/local/bin/kubectl-hello
```

`kubectl`에 사용가능한 모든 플러그인을 보려면, `kubectl plugin list` 하위 명령을 사용할 수 있다:

```shell
kubectl plugin list
```
```
다음과 같이 kubectl-호환 플러그인을 사용할 수 있다:

/usr/local/bin/kubectl-hello
/usr/local/bin/kubectl-foo
/usr/local/bin/kubectl-bar
```
```
# this command can also warn us about plugins that are
# not executable, or that are overshadowed by other
# plugins, for example
sudo chmod -x /usr/local/bin/kubectl-foo
kubectl plugin list
```
```
The following kubectl-compatible plugins are available:

/usr/local/bin/kubectl-hello
/usr/local/bin/kubectl-foo
  - warning: /usr/local/bin/kubectl-foo가 플러그인으로 식별되었지만, 실행파일은 아니다.
/usr/local/bin/kubectl-bar

error: one plugin warning was found
```

플러그인은 기존 kubectl 명령보다 복잡한 기능을 구축하는 수단으로 생각할 수 있다:

```shell
cat ./kubectl-whoami
#!/bin/bash

# 이 플러그인은 현재 선택된 컨텍스트를 기반으로 현재 사용자에 대한 정보를 출력하기 위해 'kubectl config'명령을 사용한다.
kubectl config view --template='{{ range .contexts }}{{ if eq .name "'$(kubectl config current-context)'" }}Current user: {{ .context.user }}{{ end }}{{ end }}'
```

위의 플러그인을 실행하면 KUBECONFIG 파일에서 현재 선택된 컨텍스트의 사용자가 포함된 출력된다:

```shell
# 실행가능한 파일로 만든다
sudo chmod +x ./kubectl-whoami

# PATH 경로로 이동한다
sudo mv ./kubectl-whoami /usr/local/bin

kubectl whoami
Current user: plugins-user
```

플러그인에 대한 자세한 내용은 [cli 플러그인 예제](https://github.com/kubernetes/sample-cli-plugin)를 참고하라.

{{% /capture %}}

{{% capture whatsnext %}}

[kubectl](/docs/reference/generated/kubectl/kubectl-commands/) 명령을 사용하여 시작하라.

{{% /capture %}}
