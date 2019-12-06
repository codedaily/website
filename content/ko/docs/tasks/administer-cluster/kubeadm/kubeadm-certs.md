---
title: kubeadm을 사용한 인증서 관리
content_template: templates/task
---

{{% capture overview %}}

{{< feature-state for_k8s_version="v1.15" state="stable" >}}

[kubeadm](/docs/reference/setup-tools/kubeadm/kubeadm/)으로 생성된 클라이언트 인증서는 1년 후에 만료된다. 이 페이지는 kubeadm으로 인증서 갱신을 관리하는 방법을 설명한다.

{{% /capture %}}

{{% capture prerequisites %}}

[Kubernetes의 PKI 인증서 및 요구 사항](/docs/setup/certificates/)을 숙지하자.

[Kubernetes의 PKI 인증서 및 요구 사항](/docs/setup/best-practices/certificates/)에 익숙해야한다.

{{% /capture %}}

{{% capture steps %}}

## 인증서 만료 확인

`check-expiration` 은 인증서 만료여부를 확인할 때 사용한다.

```
kubeadm alpha certs check-expiration
```

아래와 같이 출력될 것이다:

```
CERTIFICATE                EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
admin.conf                 May 15, 2020 13:03 UTC   364d            false
apiserver                  May 15, 2020 13:00 UTC   364d            false
apiserver-etcd-client      May 15, 2020 13:00 UTC   364d            false
apiserver-kubelet-client   May 15, 2020 13:00 UTC   364d            false
controller-manager.conf    May 15, 2020 13:03 UTC   364d            false
etcd-healthcheck-client    May 15, 2020 13:00 UTC   364d            false
etcd-peer                  May 15, 2020 13:00 UTC   364d            false
etcd-server                May 15, 2020 13:00 UTC   364d            false
front-proxy-client         May 15, 2020 13:00 UTC   364d            false
scheduler.conf             May 15, 2020 13:03 UTC   364d            false
```

이 명령은 kubeadm(`admin.conf`, `controller-manager.conf` and `scheduler.conf`)에서 사용하는 KUBECONFIG 파일에 포함된 클라이언트 인증서와 /etc/kubernetes/pki 폴더에 클라이언트 인증서의 만료 및 잔여 시간을 표시한다.

또한, kubeadm은 인증서가 외부에서 관리되는지 사용자에게 알려준다; 이 경우, 사용자는 수동으로 인증서 갱신 관리 및 다른 도구 사용을 관리해야 한다.

{{< warning >}}
`kubeadm`은 외부 CA가 서명 한 인증서를 관리 할 수 ​​없다.
{{< /warning >}}

{{< note >}}
kubeadm이 자동 인증서 갱신을 위해 kubelet을 구성하기 때문에 `kubelet.conf`는 위 목록에 포함되어 있지 않다.
{{< /note >}}

## 자동 인증서 갱신

`kubeadm`은 컨트롤 플레인이 [업그레이드](/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-1-15/)되면 모든 인증서가 갱신된다.

This feature is designed for addressing the simplest use cases; 
if you don't have specific requirements on certificate renewal and perform Kubernetes version upgrades regularly (less than 1 year in between each upgrade), kubeadm will take care of keeping your cluster up to date and reasonably secure.

{{< note >}}
It is a best practice to upgrade your cluster frequently in order to stay secure.
{{< /note >}}

If you have more complex requirements for certificate renewal, you can opt out from the default behavior by passing `--certificate-renewal=false` to `kubeadm upgrade apply` or to `kubeadm upgrade node`.


## Manual certificate renewal

You can renew your certificates manually at any time with the `kubeadm alpha certs renew` command.

This command performs the renewal using CA (or front-proxy-CA) certificate and key stored in `/etc/kubernetes/pki`.

{{< warning >}}
If you are running an HA cluster, this command needs to be executed on all the control-plane nodes.
{{< /warning >}}

{{< note >}}
`alpha certs renew` uses the existing certificates as the authoritative source for attributes (Common Name, Organization, SAN, etc.) instead of the kubeadm-config ConfigMap. It is strongly recommended to keep them both in sync.
{{< /note >}}

`kubeadm alpha certs renew` provides the following options:

The Kubernetes certificates normally reach their expiration date after one year.

- `--csr-only` can be used to renew certificats with an external CA by generating certificate signing requests (without actually renewing certificates in place); see next paragraph for more information.

- It's also possible to renew a single certificate instead of all.

## Renew certificates with the Kubernetes certificates API

This section provide more details about how to execute manual certificate renewal using the Kubernetes certificates API.

{{< caution >}}
These are advanced topics for users who need to integrate their organization's certificate infrastructure into a kubeadm-built cluster. If the default kubeadm configuration satisfies your needs, you should let kubeadm manage certificates instead.
{{< /caution >}}

### Set up a signer

The Kubernetes Certificate Authority does not work out of the box.
You can configure an external signer such as [cert-manager][cert-manager-issuer], or you can use the build-in signer.
The built-in signer is part of [`kube-controller-manager`][kcm].
To activate the build-in signer, you pass the `--cluster-signing-cert-file` and `--cluster-signing-key-file` arguments.

The built-in signer is part of [`kube-controller-manager`][kcm]. 

To activate the build-in signer, you must pass the `--cluster-signing-cert-file` and `--cluster-signing-key-file` flags.

If you're creating a new cluster, you can use a kubeadm [configuration file][config]:

  ```yaml
  apiVersion: kubeadm.k8s.io/v1beta2
  kind: ClusterConfiguration
  controllerManager:
    extraArgs:
      cluster-signing-cert-file: /etc/kubernetes/pki/ca.crt
      cluster-signing-key-file: /etc/kubernetes/pki/ca.key
  ```

[cert-manager-issuer]: https://cert-manager.readthedocs.io/en/latest/tutorials/ca/creating-ca-issuer.html
[kcm]: /docs/reference/command-line-tools-reference/kube-controller-manager/
[config]: https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2

### Create certificate signing requests (CSR)

You can create the certificate signing requests for the Kubernetes certificates API with `kubeadm alpha certs renew --use-api`.

If you set up an external signer such as [cert-manager][cert-manager], certificate signing requests (CSRs) are automatically approved.
Otherwise, you must manually approve certificates with the [`kubectl certificate`][certs] command.
The following kubeadm command outputs the name of the certificate to approve, then blocks and waits for approval to occur:

```shell
sudo kubeadm alpha certs renew apiserver --use-api &
```
The output is similar to this:
```
[1] 2890
[certs] certificate request "kubeadm-cert-kube-apiserver-ld526" created
```

### Approve certificate signing requests (CSR)

If you set up an external signer, certificate signing requests (CSRs) are automatically approved.

Otherwise, you must manually approve certificates with the [`kubectl certificate`][certs] command. e.g. 

```shell
kubectl certificate approve kubeadm-cert-kube-apiserver-ld526
```
The output is similar to this:
```shell
certificatesigningrequest.certificates.k8s.io/kubeadm-cert-kube-apiserver-ld526 approved
```

You can view a list of pending certificates with `kubectl get csr`.

## Renew certificates with external CA

This section provide more details about how to execute manual certificate renewal using an external CA.

To better integrate with external CAs, kubeadm can also produce certificate signing requests (CSRs).
A CSR represents a request to a CA for a signed certificate for a client.
In kubeadm terms, any certificate that would normally be signed by an on-disk CA can be produced as a CSR instead. A CA, however, cannot be produced as a CSR.

### Create certificate signing requests (CSR)

You can pass in a directory with `--csr-dir` to output the CSRs to the specified location.
If `--csr-dir` is not specified, the default certificate directory (`/etc/kubernetes/pki`) is used.
Both the CSR and the accompanying private key are given in the output. After a certificate is signed, the certificate and the private key must be copied to the PKI directory (by default `/etc/kubernetes/pki`).

A CSR represents a request to a CA for a signed certificate for a client.

You can create certificate signing requests with `kubeadm alpha certs renew --csr-only`.

Both the CSR and the accompanying private key are given in the output; you can pass in a directory with `--csr-dir` to output the CSRs to the specified location.

Certificates can be renewed with `kubeadm alpha certs renew --csr-only`.
As with `kubeadm init`, an output directory can be specified with the `--csr-dir` flag.
To use the new certificates, copy the signed certificate and private key into the PKI directory (by default `/etc/kubernetes/pki`)

A CSR contains a certificate's name, domain(s), and IPs, but it does not specify usages.

A CSR contains a certificate's name, domains, and IPs, but it does not specify usages.
It is the responsibility of the CA to specify [the correct cert usages][cert-table] when issuing a certificate.

* In `openssl` this is done with the [`openssl ca` command][openssl-ca].
* In `cfssl` you specify [usages in the config file][cfssl-usages]

After a certificate is signed using your preferred method, the certificate and the private key must be copied to the PKI directory (by default `/etc/kubernetes/pki`).

[openssl-ca]: https://superuser.com/questions/738612/openssl-ca-keyusage-extension
[cfssl-usages]: https://github.com/cloudflare/cfssl/blob/master/doc/cmd/cfssl.txt#L170
[certs]: /docs/setup/best-practices/certificates/
[cert-cas]: /docs/setup/best-practices/certificates/#single-root-ca
[cert-table]: /docs/setup/best-practices/certificates/#all-certificates

{{% /capture %}}
