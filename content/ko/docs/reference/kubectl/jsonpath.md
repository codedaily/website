---
title: JSONPath 지원
content_template: templates/concept
weight: 25
---

{{% capture overview %}}
Kubectl은 JSONPath 템플릿을 지원한다.
{{% /capture %}}

{{% capture body %}}

JSONPath 템플릿은 중괄호({})로 묶인 JSONPath 표현식으로 구성된다. Kubectl은 JSONPath 표현식을 사용하여 JSON 객체의 특정 필드를 필터링하고 출력형식을 지정한다. 원래 JSONPath 템플릿 구문 외에도 다음 함수와 구문이 유효하다:

1. JSONPath 표현식내에서 텍스트를 인용하려면 큰 따옴표를 사용하라.
2. 목록을 반복하려면, `range`, `end` 연산자를 사용하라.
3. 목록을 거꾸로 하려면, 음수 인덱스를 사용하라. 음수 인덱스는 목록을 "랩핑"하지 않으며 `-index + listLength >= 0` 이면 유효하다.

{{< note >}}

- 표현식은 항상 기본적으로 루트 개체에서 시작하므로 `$` 연산자는 선택 사항이다.

- 결과 객체는 String() 함수로 출력된다.

{{< /note >}}

JSON 입력이 주어지면:

```json
{
  "kind": "List",
  "items":[
    {
      "kind":"None",
      "metadata":{"name":"127.0.0.1"},
      "status":{
        "capacity":{"cpu":"4"},
        "addresses":[{"type": "LegacyHostIP", "address":"127.0.0.1"}]
      }
    },
    {
      "kind":"None",
      "metadata":{"name":"127.0.0.2"},
      "status":{
        "capacity":{"cpu":"8"},
        "addresses":[
          {"type": "LegacyHostIP", "address":"127.0.0.2"},
          {"type": "another", "address":"127.0.0.3"}
        ]
      }
    }
  ],
  "users":[
    {
      "name": "myself",
      "user": {}
    },
    {
      "name": "e2e",
      "user": {"username": "admin", "password": "secret"}
    }
  ]
}
```

함수                | 설명               | 예제                                                         | 결과
--------------------|---------------------------|-----------------------------------------------------------------|------------------
`text`              | 플레인 텍스트              | `kind is {.kind}`                                               | `kind is List`
`@`                 | 현재 객체                  | `{@}`                                                           | the same as input
`.` or `[]`         | 자식 연산자                | `{.kind}` or `{['kind']}`                                       | `List`
`..`                | 재귀 하강         | `{..name}`                                                      | `127.0.0.1 127.0.0.2 myself e2e`
`*`                 | 와일드카드. 모든 객체       | `{.items[*].metadata.name}`                                     | `[127.0.0.1 127.0.0.2]`
`[start:end :step]` | 서브스크립트 연산자        | `{.users[0].name}`                                              | `myself`
`[,]`               | union 연산자              | `{.items[*]['metadata.name', 'status.capacity']}`               | `127.0.0.1 127.0.0.2 map[cpu:4] map[cpu:8]`
`?()`               | 필터                    | `{.users[?(@.name=="e2e")].user.password}`                      | `secret`
`range`, `end`      | 반복 리스트              | `{range .items[*]}[{.metadata.name}, {.status.capacity}] {end}` | `[127.0.0.1, map[cpu:4]] [127.0.0.2, map[cpu:8]]`
`''`                | 인터프리티드된 문자열을 위한 따옴표  | `{range .items[*]}{.metadata.name}{'\t'}{end}`                  | `127.0.0.1      127.0.0.2`

`kubectl`와 JSONPath 표현식을 사용하는 예:

```shell
kubectl get pods -o json
kubectl get pods -o=jsonpath='{@}'
kubectl get pods -o=jsonpath='{.items[0]}'
kubectl get pods -o=jsonpath='{.items[0].metadata.name}'
kubectl get pods -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.startTime}{"\n"}{end}'
```

Windows에서는 공백이 포함된 JSONPath 템플릿을 _큰_ 따옴표로 묶어야 한다(bash에 대해 위에서 표시된 작은 따옴표가 아님). 따라서 템플릿의 모든 리터럴 주위에 작은 따옴표 또는 이스케이프 된 큰 따옴표를 사용해야 한다. 예를들어:

```cmd
C:\> kubectl get pods -o=jsonpath="{range .items[*]}{.metadata.name}{'\t'}{.status.startTime}{'\n'}{end}"
C:\> kubectl get pods -o=jsonpath="{range .items[*]}{.metadata.name}{\"\t\"}{.status.startTime}{\"\n\"}{end}"
```

{{% /capture %}}
