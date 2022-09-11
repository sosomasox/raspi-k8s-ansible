# raspi-k8s-ansible


## Prerequisites

- Ansible >= 2.9.6

&nbsp;



## デプロイ前の設定

サーバの認証情報準備のため `env` を `.env` にコピーし、パスワードを記入した後、`.env` を読み込んでください。

```
$ cat env
# サーバのパスワード
export PASSWORD=""
$ cp env .env

(.env を編集)

$ source .env
```


## デプロイ方法

### 1. 監視サーバのセットアップ

```
$ ansible-playbook --ask-become-pass -i inventroies/single-control-plane playbooks/setup4monitoring-servers.yml
```


### 2. Kubernetesクラスタを構成する各サーバのセットアップ

```
$ ansible-playbook --ask-become-pass -i inventroies/single-control-plane playbooks/setup4k8s.yml
```


### 3. kubeadmによるkubernetesクラスタの構築

```
$ ansible-playbook --ask-become-pass -i inventroies/single-control-plane playbooks/deploy4k8s.yml
```


### 4. Kubernetesクラスタの状態確認

```
ubuntu@k8s-control-plane-1:~$ kubectl get nodes
NAME                            STATUS     ROLES           AGE     VERSION
k8s-control-plane-1.home.arpa   NotReady   control-plane   3m15s   v1.24.3
k8s-control-plane-2.home.arpa   NotReady   control-plane   93s     v1.24.3
k8s-control-plane-3.home.arpa   NotReady   control-plane   94s     v1.24.3
k8s-node-1.home.arpa            NotReady   node            44s     v1.24.3
k8s-node-2.home.arpa            NotReady   node            54s     v1.24.3
k8s-node-3.home.arpa            NotReady   node            44s     v1.24.3
ubuntu@k8s-control-plane-1:~$
```

&nbsp;



## Prometheusの設定ファイルを更新したい場合

Prometheusの設定ファイルを更新したい場合、
Prometheusの設定ファイル `playbooks/roles/monitoring-servers/prometheus/templates/prometheus.yml.j2` を編集、
また、アラートルールを `playbooks/roles/monitoring-servers/prometheus/templates/rules/` 配下に設置した後で
下記のコマンドを実行し、Prometheusの設定を更新してください。  
(設定内容やアラートルールに不備がある場合、設定は更新されずに `failed` になります。)

```
$ ansible-playbook --ask-become-pass \
  -i inventroies/single-control-plane playbooks/prometheus4monitoring-servers.yml \
  --start-at-task="Create prometheus.yml"
```

&nbsp;



## Alertmanagerの設定ファイルを更新したい場合

```
$ ansible-playbook --ask-become-pass \
  -i inventroies/single-control-plane playbooks/alertmanager4monitoring-servers.yml \
  --start-at-task="Create alertmanager.yml"
```

