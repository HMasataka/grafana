# Grafana

## インストール

### 1. Helmリポジトリの追加と更新

Grafana公式のHelmチャートが格納されているリポジトリを追加  
追加したリポジトリから最新のチャート情報を取得

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### 2. Kubernetes名前空間の作成

```bash
kubectl create namespace grafana
```

### 3. Prometheusのインストール

Prometheus（メトリクス収集システム）をKubernetesクラスターにインストール

```bash
helm install prometheus -n grafana prometheus-community/kube-prometheus-stack \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.ruleSelectorNilUsesHelmValues=false \
  --set prometheus.prometheusSpec.retention=7d
```

- 設定オプション
  - `serviceMonitorSelectorNilUsesHelmValues=false`: すべてのServiceMonitorを自動検出
  - `podMonitorSelectorNilUsesHelmValues=false`: すべてのPodMonitorを自動検出
  - `ruleSelectorNilUsesHelmValues=false`: すべてのPrometheusRuleを自動検出
  - `retention=7d`: メトリクスデータの保持期間を7日に設定

Prometheusの公式リポジトリを追加する場合（まだ追加していない場合）:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 4. Lokiのインストール

Loki（ログ集約システム）をKubernetesクラスターにインストール

```bash
helm install loki -n grafana grafana/loki \
  --set minio.enabled=true \
  --set write.replicas=1 \
  --set read.replicas=1 \
  --set backend.replicas=1 \
  --set loki.auth_enabled=false \
  --set loki.commonConfig.replication_factor=1 \
  --set loki.useTestSchema=true
```

- 設定オプション
  - `minio.enabled=true`: オブジェクトストレージとしてMinIOを有効化
  - `write.replicas=1`: ログ書き込み用ポッドのレプリカ数を1に設定
  - `read.replicas=1`: ログ読み取り用ポッドのレプリカ数を1に設定
  - `backend.replicas=1`: バックエンドサービスのレプリカ数を1に設定
  - `loki.auth_enabled=false`: 認証機能を無効化（開発・テスト環境向け）
  - `loki.commonConfig.replication_factor=1`: データのレプリケーション因子を1に設定
  - `loki.useTestSchema=true`: テスト用スキーマを使用（開発環境向け）

### 5. Promtailのインストール

Kubernetesクラスター内のログを収集し、Lokiに送信

```bash
helm install promtail -n grafana grafana/promtail
```

### 6. Grafanaのインストール

Grafana（可視化ダッシュボード）をインストール  
service.type=LoadBalancer: 外部からアクセス可能なLoadBalancerサービスとして公開

```bash
helm install grafana -n grafana grafana/grafana --set service.type=LoadBalancer
```

```bash
kubectl port-forward -n grafana svc/grafana 3000:80
```

アクセス: `http://localhost:3000`

### 7. ローカル環境でのアクセス方法の変更（必要に応じて）

LoadBalancerタイプのサービスはクラウド環境でのみ機能する（ローカル環境ではNodePortなどを使用）可能性があるため、必要に応じて変更する

```bash
helm install grafana -n grafana grafana/grafana --set service.type=NodePort
```

```bash
# ノードのIPとポートを確認
kubectl get service -n grafana grafana
```

### 8. Grafana管理者パスワードの取得

Grafanaの管理者（admin）パスワードを取得

```bash
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### 9. テスト用ポッドの作成

ログ生成用のテストポッドを作成  
定期的にカウンターをログ出力するシンプルなポッド

```bash
kubectl apply -f https://k8s.io/examples/debug/counter-pod.yaml
```

### 10. Grafanaへのデータソースの追加

#### ログデータソース（Loki）の追加

GrafanaのWebインターフェースにログデータソースとしてLokiを追加

- URL: `http://loki-gateway.grafana.svc.cluster.local`

#### メトリクスデータソース（Prometheus）の追加

GrafanaのWebインターフェースにメトリクスデータソースとしてPrometheusを追加

- URL: `http://prometheus-kube-prometheus-prometheus.grafana.svc.cluster.local:9090`

または、kube-prometheus-stackをインストールした場合、GrafanaにはデフォルトでPrometheusデータソースが自動的に設定されます。
