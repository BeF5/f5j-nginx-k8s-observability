監視コンポーネントのデプロイ
####

1. Prometheusのデプロイ
====

Helmを使って設定されるDefault設定と、NIC、NSMのステータスを取得するために必要となるPrometheusの設定を追加します。

Prometheusについては以下のページを参照してください。

- `Prometheus Overview <https://prometheus.io/docs/introduction/overview/>`__

Prometheusの設定を確認します。

.. code-block:: cmdin

  # nginx-mesh-and-ingress-and-loki-scrape-config.yaml
  cat ~/observability/prep/prometheus-nginx-mesh-and-ingress-and-loki-scrape-config.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1,20

  - job_name: 'nginx-mesh-sidecars'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_container_name]
        action: keep
        regex: nginx-mesh-sidecar
      - action: labelmap
        regex: __meta_kubernetes_pod_label_nsm_nginx_com_(.+)
      - action: labeldrop
        regex: __meta_kubernetes_pod_label_nsm_nginx_com_(.+)
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: pod
  - job_name: 'nginx-plus-ingress'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_container_name]
        action: keep
        regex: nginx-plus-ingress
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: (.+)(?::\d+);(\d+)
        replacement: $1:$2
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: pod
      - action: labelmap
        regex: __meta_kubernetes_pod_label_nsm_nginx_com_(.+)
      - action: labeldrop
        regex: __meta_kubernetes_pod_label_nsm_nginx_com_(.+)
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - action: labelmap
        regex: __meta_kubernetes_pod_annotation_nsm_nginx_com_enable_(.+)
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: 'nginx_ingress_controller_upstream_server_response_latency_ms(.+)'
        target_label: __name__
        replacement: 'nginxplus_upstream_server_response_latency_ms$1'
      - source_labels: [__name__]
        regex: 'nginx_ingress_nginxplus(.+)'
        target_label: __name__
        replacement: 'nginxplus$1'
      - source_labels: [service]
        target_label: dst_service
      - source_labels: [resource_namespace]
        target_label: dst_namespace
      - source_labels: [pod_owner]
        regex: '(.+)\/(.+)'
        target_label: dst_$1
        replacement: $2
      - action: labeldrop
        regex: pod_owner
      - source_labels: [pod_name]
        target_label: dst_pod

- 1行目がNGINX Service Meshの設定の設定です
- 20行目がNGINX Ingress Controllerの設定です

| この設定を ``--set-file extraScrapeConfigs`` のオプションで指定します。
| ``kubernetes_sd_configs`` で ``Pod`` を指定し、PrometheusがPodのMetricsをScarpeします。詳細は以下のページを参照してください。

- `Prometheus CONFIGURATION kubernetes_sd_configs <https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config>`__

今回はテスト用途でPersistentVolumeを利用しないため、設定を無効にします

.. code-block:: cmdin

  cat ~/observability/prep/prometheus-addvalue.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  alertmanager:
    persistentVolume:
      enabled: false
  server:
    persistentVolume:
      enabled: false


Prometheusをデプロイします

.. code-block:: cmdin

  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
  helm upgrade --install prometheus prometheus-community/prometheus \
  -f ~/observability/prep/prometheus-addvalue.yaml \
   --set-file extraScrapeConfigs=~/observability/prep/prometheus-nginx-mesh-and-ingress-and-loki-scrape-config.yaml \
   --namespace monitor \
   --create-namespace

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Release "prometheus" does not exist. Installing it now.
  NAME: prometheus
  LAST DEPLOYED: Thu Jun 30 08:29:17 2022
  NAMESPACE: monitor
  STATUS: deployed
  REVISION: 1
  TEST SUITE: None
  NOTES:
  The Prometheus server can be accessed via port 80 on the following DNS name from within your cluster:
  prometheus-server.monitor.svc.cluster.local
  
  
  Get the Prometheus server URL by running these commands in the same shell:
    export POD_NAME=$(kubectl get pods --namespace monitor -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
    kubectl --namespace monitor port-forward $POD_NAME 9090
  #################################################################################
  ######   WARNING: Persistence is disabled!!! You will lose your data when   #####
  ######            the Server pod is terminated.                             #####
  #################################################################################
  
  
  The Prometheus alertmanager can be accessed via port 80 on the following DNS name from within your cluster:
  prometheus-alertmanager.monitor.svc.cluster.local
  
  
  Get the Alertmanager URL by running these commands in the same shell:
    export POD_NAME=$(kubectl get pods --namespace monitor -l "app=prometheus,component=alertmanager" -o jsonpath="{.items[0].metadata.name}")
    kubectl --namespace monitor port-forward $POD_NAME 9093
  #################################################################################
  ######   WARNING: Persistence is disabled!!! You will lose your data when   #####
  ######            the AlertManager pod is terminated.                       #####
  #################################################################################
  #################################################################################
  ######   WARNING: Pod Security Policy has been moved to a global property.  #####
  ######            use .Values.podSecurityPolicy.enabled with pod-based      #####
  ######            annotations                                               #####
  ######            (e.g. .Values.nodeExporter.podSecurityPolicy.annotations) #####
  #################################################################################
  
  
  The Prometheus PushGateway can be accessed via port 9091 on the following DNS name from within your cluster:
  prometheus-pushgateway.monitor.svc.cluster.local
  
  
  Get the PushGateway URL by running these commands in the same shell:
    export POD_NAME=$(kubectl get pods --namespace monitor -l "app=prometheus,component=pushgateway" -o jsonpath="{.items[0].metadata.name}")
    kubectl --namespace monitor port-forward $POD_NAME 9091
  
  For more information on running Prometheus, visit:
  https://prometheus.io/

デプロイした結果を確認します

.. code-block:: cmdin
  
  helm list -n monitor | grep prometheus

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  prometheus      monitor         1               2022-06-30 08:29:17.059609279 +0000 UTC deployed        prometheus-15.10.1      2.34.0

Podが正しく作成されていることを確認します

.. code-block:: cmdin
  
  kubectl get pod -n monitor | grep prometheus

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  prometheus-alertmanager-6b5498dfc7-l5pdj        2/2     Running   0          70s
  prometheus-kube-state-metrics-748fc7f64-scxqh   1/1     Running   0          69s
  prometheus-node-exporter-wqh9d                  1/1     Running   0          70s
  prometheus-pushgateway-b6c9dc7db-6xgb4          1/1     Running   0          69s
  prometheus-server-656659dfc6-fkwwm              2/2     Running   0          69s


2. Grafana Lokiのデプロイ
====

Helmを使って設定されるDefault設定では、LokiをデプロイするとKubernets Nodeに保存されているPodのログを取得します。
取得したログに対し、運用でログの調査が容易となるよう設定を追加します

またこのデプロイでは、Lokiの他、Promtail、Grafanaをデプロイします。

Lokiの設定パラメータについては以下のページを参照してください。

- `Promtail Scraping <https://grafana.com/docs/loki/latest/clients/promtail/stages/>`__
- `Loki LogQL <https://grafana.com/docs/loki/latest/logql/log_queries/>`__

HelmでデプロイするLokiの設定を確認します。

.. code-block:: cmdin

  cat ~/observability/prep/loki-scrape.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1, 2-8, 10-26, 28-48, 50-51

  - job_name: kubernetes-pods
    pipeline_stages:
      - cri: {}
      - json:
          expressions:
            log:
      - labels:
          log:
  
      - match:
          pipeline_name: "accesslog"
          selector: '{namespace="nginx-ingress"}  |~ "logtype##: ##accesslog"'
          stages:
          - json:
              expressions:
                log:
          - replace:
              expression: "(\"+)"
              replace: "%22"
              source: log
          - replace:
              expression: "(##)"
              replace: "\""
              source: log
          - output:
              source: log
  
      - match:
          pipeline_name: "securitylog"
          selector: '{namespace="nginx-ingress"}  |~ "logtype##: ##securitylog"'
          stages:
          - json:
              expressions:
                log:
          - replace:
              expression: "(N/A)"
              replace: "-"
              source: log
          - replace:
              expression: "(\"+)"
              replace: "%22"
              source: log
          - replace:
              expression: "(##)"
              replace: "\""
              source: log
          - output:
              source: log
  
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels:
          - __meta_kubernetes_pod_controller_name
        regex: ([0-9a-z-.]+?)(-[0-9a-f]{8,10})?
        action: replace
        target_label: __tmp_controller_name
      - source_labels:
          - __meta_kubernetes_pod_label_app_kubernetes_io_name
          - __meta_kubernetes_pod_label_app
          - __tmp_controller_name
          - __meta_kubernetes_pod_name
        regex: ^;*([^;]+)(;.*)?$
        action: replace
        target_label: app
      - source_labels:
          - __meta_kubernetes_pod_label_app_kubernetes_io_component
          - __meta_kubernetes_pod_label_component
        regex: ^;*([^;]+)(;.*)?$
        action: replace
        target_label: component
      - action: replace
        source_labels:
        - __meta_kubernetes_pod_node_name
        target_label: node_name
      - action: replace
        source_labels:
        - __meta_kubernetes_namespace
        target_label: namespace
      - action: replace
        replacement: $1
        separator: /
        source_labels:
        - namespace
        - app
        target_label: job
      - action: replace
        source_labels:
        - __meta_kubernetes_pod_name
        target_label: pod
      - action: replace
        source_labels:
        - __meta_kubernetes_pod_container_name
        target_label: container
      - action: replace
        replacement: /var/log/pods/*$1/*.log
        separator: /
        source_labels:
        - __meta_kubernetes_pod_uid
        - __meta_kubernetes_pod_container_name
        target_label: __path__
      - action: replace
        regex: true/(.*)
        replacement: /var/log/pods/*$1/*.log
        separator: /
        source_labels:
        - __meta_kubernetes_pod_annotationpresent_kubernetes_io_config_hash
        - __meta_kubernetes_pod_annotation_kubernetes_io_config_hash
        - __meta_kubernetes_pod_container_name
        target_label: __path__

- 50-51行目で ``kubernetes_sd_configs`` の ``pod`` を指定し、各Nodeに記録されているPodのログを取得する設定となっています。50行目以降がHelmでデプロイする際のデフォルトの設定となります
- 2行目の ``cri`` で取得したログを、3-8行目で json でパースし、log 部分を抽出します
- 10-26行目は、8行目までで抽出した log の内容に対し、 match ステージでNGINXの ``accesslog`` の条件を指定しログを抽出します
- 28-48行目は、10-26行目同様に match ステージでNAP WAFの ``securitylog`` の条件を指定しログを抽出します

参考の追加設定としてSyslog Serverの設定を追加します

.. code-block:: cmdin

  cat ~/observability/prep/loki-scrape-addvalue.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  # this is extraScrapeCOnfig
  - job_name: syslog
    syslog:
      listen_address: 0.0.0.0:1514
      labels:
        job: "syslog"
    relabel_configs:
      - source_labels: ['__syslog_message_hostname']
        target_label: 'host'

その他Lokiの設定パラメータは以下を参照してください

- `GitHub helm-charts/charts/loki-stack/ <https://github.com/grafana/helm-charts/tree/main/charts/loki-stack>`__
- `GitHub helm-charts/charts/loki-stack/values.yaml <https://github.com/grafana/helm-charts/blob/main/charts/loki-stack/values.yaml>`__

Lokiをデプロイします

.. code-block:: cmdin

  helm repo add grafana https://grafana.github.io/helm-charts
  helm upgrade --install loki grafana/loki-stack -n monitor \
   --set grafana.enabled=true \
   --set-file promtail.config.snippets.extraScrapeConfigs=~/observability/prep/loki-scrape-addvalue.yaml \
   --set-file promtail.config.snippets.scrapeConfigs=~/observability/prep/loki-scrape.yaml 

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Release "loki" does not exist. Installing it now.
  W0630 10:11:21.164451  201978 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
  W0630 10:11:21.167201  201978 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
  W0630 10:11:21.169425  201978 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
  W0630 10:11:21.345337  201978 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
  W0630 10:11:21.346284  201978 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
  W0630 10:11:21.346657  201978 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
  NAME: loki
  LAST DEPLOYED: Thu Jun 30 10:11:19 2022
  NAMESPACE: monitor
  STATUS: deployed
  REVISION: 1
  NOTES:
  The Loki stack has been deployed to your cluster. Loki can now be added as a datasource in Grafana.
  
  See http://docs.grafana.org/features/datasources/loki/ for more detail.

デプロイした結果を確認します

.. code-block:: cmdin
  
  helm list -n monitor | grep loki

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  loki            monitor         1               2022-06-30 10:11:19.749832951 +0000 UTC deployed        loki-stack-2.6.5        v2.4.2

Podが正しく作成されていることを確認します

.. code-block:: cmdin
  
  kubectl get pod -n monitor | grep loki

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  loki-0                                          1/1     Running   0          2m19s
  loki-grafana-668cc48b7f-4t5cq                   2/2     Running   0          2m19s
  loki-promtail-gcqck                             1/1     Running   0          2m19s
  loki-promtail-xfznr                             1/1     Running   0          2m19s

3. Jaegerのデプロイ
====

| 動作確認のため、all-in-one のJaegerをデプロイします。
| HelmでデプロイするJaegerの設定を確認します。

Jaegerについては以下を参照してください。

- `JAEGER Getting Started <https://www.jaegertracing.io/docs/next-release/getting-started/>`__


.. code-block:: cmdin

  cat ~/observability/prep/jaeger-addvalues.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 6-7

  provisionDataStore:
    cassandra: false
    elasticsearch: false
    kafka: false
  
  allInOne:
    enabled: true
  #  image: jaegertracing/all-in-one
  #  tag: 1.29.0
    ingress:
      enabled: false
  
  collector:
    enabled: false
  query:
    enabled: false
  agent:
    enabled: false

- 6-7行目で ``allInOne`` の形式でデプロイすることを指定し、その他パラメータでふおうな設定を解除します

Jaegerの設定パラメータについては以下のページを参照してください。

- `GitHub helm-charts/charts/jaeger/ <https://github.com/jaegertracing/helm-charts/tree/main/charts/jaeger>`__
- `GitHub helm-charts/charts/jaeger/values.yaml <https://github.com/jaegertracing/helm-charts/tree/main/charts/jaeger/values.yaml>`__

Jaegerをデプロイします

.. code-block:: cmdin

  helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
  helm upgrade --install jaeger jaegertracing/jaeger -n monitor -f jaeger-addvalues.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  
  Release "jaeger" does not exist. Installing it now.
  NAME: jaeger
  LAST DEPLOYED: Thu Jun 30 10:37:49 2022
  NAMESPACE: monitor
  STATUS: deployed
  REVISION: 1
  TEST SUITE: None
  NOTES:
  ###################################################################
  ### IMPORTANT: Ensure that storage is explicitly configured     ###
  ### Default storage options are subject to change.              ###
  ###                                                             ###
  ### IMPORTANT: The use of <component>.env: {...} is deprecated. ###
  ### Please use <component>.extraEnv: [] instead.                ###
  ###################################################################
  
  You can log into the Jaeger Query UI here:
  
    export POD_NAME=$(kubectl get pods --namespace monitor -l "app.kubernetes.io/instance=jaeger,app.kubernetes.io/com                                    ponent=query" -o jsonpath="{.items[0].metadata.name}")
    echo http://127.0.0.1:8080/
    kubectl port-forward --namespace monitor $POD_NAME 8080:16686

デプロイした結果を確認します

.. code-block:: cmdin
  
  helm list -n monitor | grep jaeger

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  jaeger          monitor         1               2022-06-30 10:37:49.835438814 +0000 UTC deployed        jaeger-0.56.8           1.30.0

Podが正しく作成されていることを確認します

.. code-block:: cmdin
  
  kubectl get pod,svc -n monitor | grep jaeger

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  pod/jaeger-7896dffdb6-gmlp8                         1/1     Running   0          77s
  service/jaeger-agent                    ClusterIP   None             <none>        5775/UDP,5778/TCP,6831/UDP,6832/UDP      82s
  service/jaeger-collector                ClusterIP   None             <none>        9411/TCP,14250/TCP,14267/TCP,14268/TCP   82s
  service/jaeger-query                    ClusterIP   None             <none>        16686/TCP,16685/TCP                      82s




Tips1. Helmでパラメータを指定する際の主なデバッグ方法
====

| Tips2で紹介の通り、デプロイしたHelmの状態を確認することができます。
| パラメータを指定した場合には以下のような手順に沿って、調査することが有効です

- 1. ドキュメントを参照する。取得するHelm Chartや、Chartが参照するGitHubの内容を確認します

  - `Prometheus helm-charts <https://prometheus-community.github.io/helm-charts/>`__
  - `GitHub helm-charts/prometheus <https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus>`__
  - `GitHub helm-charts/prometheus values.yaml <https://github.com/prometheus-community/helm-charts/blob/main/charts/prometheus/values.yaml>`__

- 2. デフォルトの設定でデプロイする
- 3. デプロイした内容を確認する。helm get コマンドを用いて状態を確認できます

  .. code-block:: cmdin
  
    $ helm get -h
    
    This command consists of multiple subcommands which can be used to
    get extended information about the release, including:
    
    - The values used to generate the release
    - The generated manifest file
    - The notes provided by the chart of the release
    - The hooks associated with the release
    
    Usage:
      helm get [command]
    
    Available Commands:
      all         download all information for a named release
      hooks       download all hooks for a named release
      manifest    download the manifest for a named release
      notes       download the notes for a named release
      values      download the values file for a named release

- 4. 1. や 3. の内容を元に設定ファイルパラメータを記述する
- 5. 4. で記述した内容が正しく反映されることを3. の手順を参考に確認する

  - ``-f`` で指定することで、ファイルの形式でオプションパラメータを指定することができます
  - ``--set`` で、パラメータの値を個別に指定することができます
  - ``--set-file`` で、対象のパラメータに対し、ファイル形式で値を指定することができます

- 6. 意図した動作となっていることを確認する

Tips2. Helmでデプロイするリソースの詳細
====

Helmを使ってデプロイしたPrometheusについて、どのようなステータスとなっている確認する方法紹介します

以下コマンドを実行し、出力結果を確認します

.. code-block:: cmdin
  
  helm get all prometheus -n monitor | less


.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1-7,17,36-37
  
  NAME: prometheus
  LAST DEPLOYED: Thu Jun 30 08:38:08 2022
  NAMESPACE: monitor
  STATUS: deployed
  REVISION: 1
  TEST SUITE: None
  USER-SUPPLIED VALUES:
  alertmanager:
    persistentVolume:
      enabled: false
  extraScrapeConfigs: |+
    - job_name: 'nginx-mesh-sidecars'
      kubernetes_sd_configs:
        - role: pod
  **省略**
  
  COMPUTED VALUES:
  alertRelabelConfigs: null
  alertmanager:
  **省略** 

    persistentVolume:
      accessModes:
      - ReadWriteOnce
      annotations: {}
      enabled: false
  **省略**

  extraScrapeConfigs: |+
    - job_name: 'nginx-mesh-sidecars'
      kubernetes_sd_configs:
        - role: pod
      relabel_configs:
  **省略**

  HOOKS:
  MANIFEST:
  ---
  # Source: prometheus/charts/kube-state-metrics/templates/serviceaccount.yaml
  apiVersion: v1
  kind: ServiceAccount

- ``helm get all`` を指定すると全ての情報を確認することができます
- helm 実行時に入力した情報は ``USER-SUPPLIED VALUES`` に表示されます
- ``COMPUTED VALUES`` に実際に適用される情報が表示されますので、指定した値が意図した通りとなっているか確認してください


