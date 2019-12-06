---
title: kubectl 설치 및 설정
content_template: templates/task
weight: 10
card:
  name: tasks
  weight: 20
  title: Install kubectl
---

{{% capture overview %}}
Kubernetes 커맨드라인 도구인 [kubectl](/docs/user-guide/kubectl/)을 사용하면 Kubernetes 클러스터에 대해 명령을 실행할 수 있다. kubectl을 사용하여 어플리케이션을 배포하고 클러스터 리소스를 검사 및 관리하며 로그를 볼 수 있다. kubectl 작업의 전체 목록은 [kubectl 개요](/docs/reference/kubectl/overview/)를 참조하라. 
{{% /capture %}}

{{% capture prerequisites %}}
클러스터의 마이너버전 차이가 나는 kubectl 버전을 사용해야 한다. 예를 들어, v1.2 클라이언트는 v1.1, v1.2 및 v1.3 마스터와 함께 사용해야 한다. 최신 버전의 kubectl을 사용하면 예기치 않은 문제를 피할 수 있다.
{{% /capture %}}

{{% capture steps %}}

## 리눅스에서 kubectl 설치

### Linux에서 curl을 사용하여 kubectl 바이너리 설치

1. 다음 명령으로 최신 릴리스를 다운로드하라:

    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
    ```

    특정 버전을 다운로드하려면 명령의 `$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)` 부분을 특정 버전으로 바꿔라.

    예를 들어 Linux에서 버전 {{< param "fullversion" >}} 을 다운로드하려면 다음을 입력하라:
    
    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/{{< param "fullversion" >}}/bin/linux/amd64/kubectl
    ```

2. kubectl 바이너리를 실행 가능하게 만든다.

    ```
    chmod +x ./kubectl
    ```

3. 바이너리를 PATH로 이동한다.

    ```
    sudo mv ./kubectl /usr/local/bin/kubectl
    ```
4. 설치한 버전이 최신인지 확인:

    ```
    kubectl version
    ```

### 기본 패키지 관리를 사용하여 설치

{{< tabs name="kubectl_install" >}}
{{< tab name="Ubuntu, Debian or HypriotOS" codelang="bash" >}}
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
{{< /tab >}}
{{< tab name="CentOS, RHEL or Fedora" codelang="bash" >}}cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubectl
{{< /tab >}}
{{< /tabs >}}


### snap으로 설치

[snap](https://snapcraft.io/docs/core/install) 패키지 관리자를 지원하는 Ubuntu 또는 다른 Linux 배포판 인 경우 kubectl을 [snap](https://snapcraft.io/) 어플리케이션으로 사용할 수 있다.

1. 스냅 사용자로 전환하고 설치 명령을 실행하라:

    ```
    sudo snap install kubectl --classic
    ```

2. 설치한 버전이 최신인지 확인:

    ```
    kubectl version
    ```

## macOS에 kubectl 설치

### macOS에서 curl을 사용하여 kubectl 바이너리 설치

1. 최신 릴리스 다운로드:

    ```		 
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
    ```

    특정 버전을 다운로드하려면 명령의 `$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)` 부분을 특정 버전으로 바꿔라.

    예를 들어, macOS에서 버전 {{< param "fullversion" >}}을 다운로드하려면 다음을 입력하라:
		  
    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/{{< param "fullversion" >}}/bin/darwin/amd64/kubectl
    ```

2. kubectl 바이너리를 실행 가능하게 만든다.

    ```
    chmod +x ./kubectl
    ```

3. 바이너리를 PATH로 이동한다.

    ```
    sudo mv ./kubectl /usr/local/bin/kubectl
    ```
4. 설치한 버전이 최신인지 확인:

    ```
    kubectl version
    ```

### macOS에서 Homebrew를 사용하여 설치

macOS를 사용 중이고 [Homebrew](https://brew.sh/) 패키지 관리자를 사용하는 경우 Homebrew와 함께 kubectl을 설치할 수 있다.

1. 설치 명령을 실행하라:

    ```
    brew install kubernetes-cli
    ```

2. 설치한 버전이 최신인지 확인:

    ```
    kubectl version
    ```

### macOS에서 Macports를 사용하여 설치

macOS를 사용 중이고 [Macports](https://macports.org/) 패키지 관리자를 사용하는 경우 Macports와 함께 kubectl을 설치할 수 있다.

1. 설치 명령을 실행하라:

    ```
    sudo port selfupdate
    sudo port install kubectl
    ```
    
2. 설치한 버전이 최신인지 확인:

    ```
    kubectl version
    ```

## 윈도우에서 kubectl 설치

### Windows에서 curl을 사용하여 kubectl 바이너리 설치

1. [이 링크](https://storage.googleapis.com/kubernetes-release/release/{{< param "fullversion" >}}/bin/windows/amd64/kubectl.exe) 에서 최신 릴리스 {{< param "fullversion" >}}을 다운로드하라.

    또는 `curl`이 설치되어 있으면 다음 명령을 사용하라:

    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/{{< param "fullversion" >}}/bin/windows/amd64/kubectl.exe
    ```

    안정적인 최신 버전 (예 : 스크립팅)을 찾으려면 [https://storage.googleapis.com/kubernetes-release/release/stable.txt](https://storage.googleapis.com/kubernetes-release/release/stable.txt)를 참조하라.

2. 바이너리를 PATH로 추가 한다.
3. `kubectl` 버전이 다운로드한 버전과 같은지 확인:

    ```
    kubectl version
    ```
{{< note >}}
[Windows 용 Docker]((https://docs.docker.com/docker-for-windows/#kubernetes))는 자체 버전의 `kubectl`을 PATH에 추가한다. Docker를 이전에 설치한 경우, Docker 설치 프로그램에서 추가한 항목 앞에 PATH 항목을 배치하거나 Docker의 `kubectl`을 제거해야 할 수도 있다.
{{< /note >}}

### PSGallery에서 Powershell로 설치

Windows와 [Powershell Gallery](https://www.powershellgallery.com/) 패키지 관리자를 사용하는 경우, Powershell을 사용하여 kubectl을 설치하고 업데이트 할 수 있다.

1. 설치 명령을 실행하라(`DownloadLocation`을 지정하라):

    ```
    Install-Script -Name install-kubectl -Scope CurrentUser -Force
    install-kubectl.ps1 [-DownloadLocation <path>]
    ```
    
    {{< note >}}
    `DownloadLocation`을 지정하지 않으면, `kubectl`이 사용자의 임시 디렉토리에 설치된다.
    {{< /note >}}
    
    설치 프로그램은 `$HOME/.kube` 폴더를 만들고 구성 파일을 만들도록 지시한다.

2. 설치한 버전이 최신인지 확인:

    ```
    kubectl version
    ```

    {{< note >}}
    1단계에 나열된 두 명령을 다시 실행하여 설치 업데이트를 수행한다.
    {{< /note >}}

### Chocolatey 또는 Scoop을 사용하여 Windows에 설치.

To install kubectl on Windows you can use either [Chocolatey](https://chocolatey.org) package manager or [Scoop](https://scoop.sh) command-line installer.
{{< tabs name="kubectl_win_install" >}}
{{% tab name="choco" %}}

    choco install kubernetes-cli

{{% /tab %}}
{{% tab name="scoop" %}}

    scoop install kubectl

{{% /tab %}}
{{< /tabs >}}
2. Test to ensure the version you installed is up-to-date:

    ```
    kubectl version
    ```

3. Navigate to your home directory:

    ```
    cd %USERPROFILE%
    ```
4. Create the `.kube` directory:

    ```
    mkdir .kube
    ```

5. Change to the `.kube` directory you just created:

    ```
    cd .kube
    ```

6. Configure kubectl to use a remote Kubernetes cluster:

    ```
    New-Item config -type file
    ```
    
    {{< note >}}Edit the config file with a text editor of your choice, such as Notepad.{{< /note >}}

## Download as part of the Google Cloud SDK

You can install kubectl as part of the Google Cloud SDK.

1. Install the [Google Cloud SDK](https://cloud.google.com/sdk/).
2. Run the `kubectl` installation command:

    ```
    gcloud components install kubectl
    ```
    
3. Test to ensure the version you installed is up-to-date:

    ```
    kubectl version
    ```

## Verifying kubectl configuration 

In order for kubectl to find and access a Kubernetes cluster, it needs a [kubeconfig file](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/), which is created automatically when you create a cluster using `kube-up.sh` or successfully deploy a Minikube cluster. By default, kubectl configuration is located at `~/.kube/config`.

Check that kubectl is properly configured by getting the cluster state:

```shell
kubectl cluster-info
```
If you see a URL response, kubectl is correctly configured to access your cluster.

If you see a message similar to the following, kubectl is not configured correctly or is not able to connect to a Kubernetes cluster.

```shell
The connection to the server <server-name:port> was refused - did you specify the right host or port?
```

For example, if you are intending to run a Kubernetes cluster on your laptop (locally), you will need a tool like Minikube to be installed first and then re-run the commands stated above.

If kubectl cluster-info returns the url response but you can't access your cluster, to check whether it is configured properly, use:

```shell
kubectl cluster-info dump
```

## Optional kubectl configurations

### Enabling shell autocompletion

kubectl provides autocompletion support for Bash and Zsh, which can save you a lot of typing.

Below are the procedures to set up autocompletion for Bash (including the difference between Linux and macOS) and Zsh.

{{< tabs name="kubectl_autocompletion" >}}

{{% tab name="Bash on Linux" %}}

### Introduction

The kubectl completion script for Bash can be generated with the command `kubectl completion bash`. Sourcing the completion script in your shell enables kubectl autocompletion.

However, the completion script depends on [**bash-completion**](https://github.com/scop/bash-completion), which means that you have to install this software first (you can test if you have bash-completion already installed by running `type _init_completion`).

### Install bash-completion

bash-completion is provided by many package managers (see [here](https://github.com/scop/bash-completion#installation)). You can install it with `apt-get install bash-completion` or `yum install bash-completion`, etc.

The above commands create `/usr/share/bash-completion/bash_completion`, which is the main script of bash-completion. Depending on your package manager, you have to manually source this file in your `~/.bashrc` file.

To find out, reload your shell and run `type _init_completion`. If the command succeeds, you're already set, otherwise add the following to your `~/.bashrc` file:

```shell
source /usr/share/bash-completion/bash_completion
```

Reload your shell and verify that bash-completion is correctly installed by typing `type _init_completion`.

### Enable kubectl autocompletion

You now need to ensure that the kubectl completion script gets sourced in all your shell sessions. There are two ways in which you can do this:

- Source the completion script in your `~/.bashrc` file:

    ```shell
    echo 'source <(kubectl completion bash)' >>~/.bashrc
    ```

- Add the completion script to the `/etc/bash_completion.d` directory:

    ```shell
    kubectl completion bash >/etc/bash_completion.d/kubectl
    ```

{{< note >}}
bash-completion sources all completion scripts in `/etc/bash_completion.d`.
{{< /note >}}

Both approaches are equivalent. After reloading your shell, kubectl autocompletion should be working.

{{% /tab %}}


{{% tab name="Bash on macOS" %}}


### Introduction

The kubectl completion script for Bash can be generated with `kubectl completion bash`. Sourcing this script in your shell enables kubectl completion.

However, the kubectl completion script depends on [**bash-completion**](https://github.com/scop/bash-completion) which you thus have to previously install.

{{< warning>}}
there are two versions of bash-completion, v1 and v2. V1 is for Bash 3.2 (which is the default on macOS), and v2 is for Bash 4.1+. The kubectl completion script **doesn't work** correctly with bash-completion v1 and Bash 3.2. It requires **bash-completion v2** and **Bash 4.1+**. Thus, to be able to correctly use kubectl completion on macOS, you have to install and use Bash 4.1+ ([*instructions*](https://itnext.io/upgrading-bash-on-macos-7138bd1066ba)). The following instructions assume that you use Bash 4.1+ (that is, any Bash version of 4.1 or newer).
{{< /warning >}}


### Install bash-completion

{{< note >}}
As mentioned, these instructions assume you use Bash 4.1+, which means you will install bash-completion v2 (in contrast to Bash 3.2 and bash-completion v1, in which case kubectl completion won't work).
{{< /note >}}

You can test if you have bash-completion v2 already installed with `type _init_completion`. If not, you can install it with Homebrew:

```shell
brew install bash-completion@2
```

As stated in the output of this command, add the following to your `~/.bashrc` file:

```shell
export BASH_COMPLETION_COMPAT_DIR="/usr/local/etc/bash_completion.d"
[[ -r "/usr/local/etc/profile.d/bash_completion.sh" ]] && . "/usr/local/etc/profile.d/bash_completion.sh"
```

Reload your shell and verify that bash-completion v2 is correctly installed with `type _init_completion`.

### Enable kubectl autocompletion

You now have to ensure that the kubectl completion script gets sourced in all your shell sessions. There are multiple ways to achieve this:

- Source the completion script in your `~/.bashrc` file:

    ```shell
    echo 'source <(kubectl completion bash)' >>~/.bashrc

    ```

- Add the completion script to the `/usr/local/etc/bash_completion.d` directory:

    ```shell
    kubectl completion bash >/usr/local/etc/bash_completion.d/kubectl
    ```

- If you installed kubectl with Homebrew (as explained [above](#install-with-homebrew-on-macos)), then the kubectl completion script should already be in `/usr/local/etc/bash_completion.d/kubectl`. In that case, you don't need to do anything.

{{< note >}}
the Homebrew installation of bash-completion v2 sources all the files in the `BASH_COMPLETION_COMPAT_DIR` directory, that's why the latter two methods work.
{{< /note >}}

In any case, after reloading your shell, kubectl completion should be working.
{{% /tab %}}

{{% tab name="Zsh" %}}

The kubectl completion script for Zsh can be generated with the command `kubectl completion zsh`. Sourcing the completion script in your shell enables kubectl autocompletion.

To do so in all your shell sessions, add the following to your `~/.zshrc` file:

```shell
source <(kubectl completion zsh)
```

After reloading your shell, kubectl autocompletion should be working.

If you get an error like `complete:13: command not found: compdef`, then add the following to the beginning of your `~/.zshrc` file:

```shell
autoload -Uz compinit
compinit
```
{{% /tab %}}
{{< /tabs >}}

{{% /capture %}}

{{% capture whatsnext %}}
* [Install Minikube](/docs/tasks/tools/install-minikube/)
* See the [getting started guides](/docs/setup/) for more about creating clusters. 
* [Learn how to launch and expose your application.](/docs/tasks/access-application-cluster/service-access-application-cluster/)
* If you need access to a cluster you didn't create, see the [Sharing Cluster Access document](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/).
* Read the [kubectl reference docs](/docs/reference/kubectl/kubectl/)
{{% /capture %}}
