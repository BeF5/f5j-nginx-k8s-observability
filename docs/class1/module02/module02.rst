NGINX NIC / NSM のセットアップ
####

1. 事前セットアップ、HELMのインストール
====

必要なファイルを取得します。

.. code-block:: cmdin
  
  cd ~/
  git clone https://github.com/BeF5/f5j-nsm-lab.git
  cd ~/f5j-nsm-lab/prep

NSMテスト用のNamespaceを作成します

.. code-block:: cmdin
  
  # cd ~/f5j-nsm-lab/prep
  kubectl apply -f nsm-demo-ns.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  namespace/prod created
  namespace/staging created
  namespace/legacy created


こちらのラボでは、HELMを使って環境をセットアップします。
HELMをinstallします。

.. code-block:: cmdin

  curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
  sudo apt-get update
  sudo apt-get install helm

正しくHELMがインストールされたことを確認します

.. code-block:: cmdin

  helm version

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  
  version.BuildInfo{Version:"v3.9.0", GitCommit:"7ceeda6c585217a19a1131663d8cd1f7d641b2a7", GitTreeState:"clean", GoVersion:"go1.17.5"}


必要なファイルを取得します

.. code-block:: cmdin
  
  cd ~/
  git clone https://github.com/BeF5/f5j-nginx-observability-lab.git

2. NSMのセットアップ
====

必要なファイルを取得します

.. code-block:: cmdin

  cd ~/
  git clone https://github.com/nginxinc/nginx-service-mesh --branch v1.4.1
  cd ~/nginx-service-mesh

取得した内容が意図したVersionであることを確認します

.. code-block:: cmdin

  ## cd ~/nginx-service-mesh
  git show -s

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  commit c605618f6226926c3a0c2b0984a44f8844ae1d75 (HEAD, tag: v1.4.1, origin/main, origin/HEAD)
  Merge: 263f119 7ee8afd
  Author: Pamme <pammecrandall@yahoo.com>
  Date:   Thu May 26 12:42:52 2022 -0600
  
      Merge pull request #68 from nginxinc/release-1.4.1
  
      Helm release - 1.4.1

| HelmでNSMをセットアップする際に用いる、パラメータの内容を確認します。
| Defaultの値は `GitHub nginx-service-mesh/helm-chart/values.yaml <https://github.com/nginxinc/nginx-service-mesh/blob/main/helm-chart/values.yaml>`__ の内容を確認してください。

.. code-block:: cmdin

  cat ~/f5j-nginx-observability-lab/prep/helm/nsm-values.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 11,20,24,28,32,40,44,48,51,76

  # Environment to deploy the mesh into.
  # Valid values: kubernetes, openshift
  environment: "kubernetes"
  
  # Enable UDP traffic proxying (beta). Linux kernel 4.18 or greater is required.
  enableUDP: false
  
  # Deploy Grafana as a part of the NGINX Service Mesh.
  # Note: This configurable will be removed in version 1.5
  # Valid values: true, false
  deployGrafana: false
  
  # NGINX log format.
  # Valid values: default, json
  nginxLogFormat: "json"
  
  # NGINX load balancing method.
  # Valid values: [least_conn, least_time, least_time last_byte, least_time last_byte inflight,
  # random, random two, random two least_conn, random two least_time, random two least_time=last_byte, round_robin]
  nginxLBMethod: "round_robin"
  
  # The address of a Prometheus server deployed in your Kubernetes cluster.
  # Address should be in the format <service-name>.<namespace>:<service-port>.
  prometheusAddress: "prometheus-server.monitor:80"
  
  # NGINX Service Mesh auto-injection settings.
  autoInjection:
    disable: true
  
    # Enable automatic sidecar injection for specific namespaces.
    # Must be used with "disable".
    enabledNamespaces: [ staging , prod ]
  
  # NGINX Service Mesh tracing settings.
  # Cannot be set when telemetry is set.
  # If deploying with tracing, make sure the telemetry object is set to {}.
  tracing:
    # Disable tracing for all services.
    # Note: This configurable will be removed in version 1.5
    disable: false
  
    # The address of a tracing server deployed in your Kubernetes cluster.
    # Address should be in the format <service-name>.<namespace>:<service_port>.
    address: "jaeger-agent.monitor:6831"
  
    # The tracing backend that you want to use.
    # Valid values: datadog, jaeger, zipkin
    backend: "jaeger"
  
    # The sample rate to use for tracing. Float between 0 and 1.
    sampleRate: 1
  
  # Mutual TLS settings. See https://docs.nginx.com/nginx-service-mesh/guides/secure-traffic-mtls for more info.
  mtls:
    # mTLS mode for pod-to-pod communication.
    # Valid values: off, permissive, strict
    mode: "strict"
  
    # Use persistent storage; "on" assumes that a StorageClass exists.
    # Valid values: on, off
    persistentStorage: "off"

- | 11行目でGrafanaのDeployを無効に、40行目でJaegerのDeployを無効にします
  | (コメント欄に記載の通り、Jaegerのパラメータは NSM v1.5で削除されます)
- 24行目でPrometheus、44行目・48行目でJaegerの設定を指定します
- この例ではTraceの情報の結果を容易に確認するため、SampleRate 1 と指定します

NSMをデプロイします

.. code-block:: cmdin

  cd ~/nginx-service-mesh/helm-chart
  cp ~/f5j-nginx-observability-lab/prep/helm/nsm-values.yaml .
  helm upgrade --install nsm -f nsm-values.yaml . \
   --namespace nginx-mesh \
   --create-namespace

- -f オプションで先程のファイルをしていすることにより、Helmのデプロイのパラメータとして付与します
- --namespace オプションでHelmで展開するNamespaceを指定します
- --create-namespace により対象のNamespaceが存在しない場合、Helmコマンド実行時に作成します

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Release "nsm" does not exist. Installing it now.
  NAME: nsm
  LAST DEPLOYED: Thu Jun 30 06:46:04 2022
  NAMESPACE: nginx-mesh
  STATUS: deployed
  REVISION: 1
  TEST SUITE: None
  NOTES:
  NGINX Service Mesh has been installed. Ensure all NGINX Service Mesh Pods are in the Ready state before deploying yo                              ur apps.

デプロイの結果を確認します

.. code-block:: cmdin

  helm list -n nginx-mesh

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
  nsm     nginx-mesh      1               2022-06-30 06:46:04.963589733 +0000 UTC deployed        nginx-service-mesh-0.4.1        1.4.1

Podが正しく作成され、以下のようになることを確認してください

.. code-block:: cmdin

  kubectl get pod -n nginx-mesh

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                                  READY   STATUS    RESTARTS   AGE
  nats-server-cf97cf4f4-9ggnq           2/2     Running   0          92s
  nginx-mesh-api-5c99b4df77-8kmw9       1/1     Running   0          92s
  nginx-mesh-metrics-5d856c4dfc-fhw7d   1/1     Running   0          92s
  spire-agent-x4smj                     1/1     Running   0          93s
  spire-server-66c596b85c-gfkz2         2/2     Running   0          92s

3. NICのセットアップ
====

必要なファイルを取得します

ファイルを取得します

.. code-block:: cmdin

  cd ~/
  git clone https://github.com/nginxinc/kubernetes-ingress.git --branch v2.2.2
  cd ~/kubernetes-ingress/

取得した内容が意図したVersionであることを確認します

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/
  git show -s

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  commit a88b7fe6dbde5df79593ac161749afc1e9a009c6 (HEAD, tag: v2.2.2)
  Author: Luca Comellini <luca.com@gmail.com>
  Date:   Mon May 23 12:56:33 2022 -0700
  
      Release 2.2.2 (#2711)


NAP DoS の Arbitator をデプロイします

.. code-block:: cmdin

  cd ~/kubernetes-ingress/deployments/helm-chart-dos-arbitrator
  helm upgrade --install appdos-arbitrator . \
   --namespace nginx-ingress \
   --create-namespace

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Release "appdos-arbitrator" does not exist. Installing it now.
  NAME: appdos-arbitrator
  LAST DEPLOYED: Tue Jun 28 12:32:37 2022
  NAMESPACE: nginx-ingress
  STATUS: deployed
  REVISION: 1
  TEST SUITE: None

デプロイの結果を確認します

.. code-block:: cmdin

  helm list -n nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                                   APP VERSION
  appdos-arbitrator       nginx-ingress   1               2022-06-28 12:32:37.157945967 +0000 UTC deployed        nginx-appprotect-dos-arbitrator-0.1.0   1.1.0

Podが正しく作成され、以下のようになることを確認してください

.. code-block:: cmdin

  kubectl get pod -n nginx-ingress | grep dos

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  appdos-arbitrator-nginx-appprotect-dos-arbitrator-844bdf64qjw9l   1/1     Running   0          23s

| NICのコンテナイメージを作成します。
| ここでは実行するコマンドを記載します。詳細は 手順: `NIC環境のセットアップ <https://f5j-nginx-ingress-controller-lab1.readthedocs.io/en/latest/class1/module2/module2.html#id1>`__ を参照ください
| (イメージのデプロイには5分程度かかる場合があります)

.. code-block:: cmdin

  cd ~/kubernetes-ingress/
  cp ~/nginx-repo* .
  ls nginx-repo.*
  make debian-image-nap-dos-plus PREFIX=registry.example.com/root/nic/nginxplus-ingress-nap-dos TARGET=container TAG=2.2.2
  docker login registry.example.com
   Username: root       << 左の文字列を入力
   Password: password   << 左の文字列を入力
  docker push registry.example.com/root/nic/nginxplus-ingress-nap-dos:2.2.2

NICをデプロイします。

| NSMを利用するアプリケーションへの通信を制御する ``nic1`` と、
| その他管理コンポーネントなどへの通信を制御する ``nic2`` をデプロイします。

| ``nic1`` で指定するパラメータの内容を確認します。
| Defaultの値は `GitHub kubernetes-ingress/deployments/helm-chart/values.yaml <https://github.com/nginxinc/kubernetes-ingress/blob/main/deployments/helm-chart/values.yaml>`__ の内容を確認してください。

.. code-block:: cmdin

  cat ~/f5j-nginx-observability-lab/prep/helm/nic1-addvalue.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1-3, 6-9, 12-13, 16-17, 19, 47-51, 56-67 

  nginxServiceMesh:
    enable: true
    enableEgress: true
  
  controller:
    nginxplus: true
    image:
      repository: registry.example.com/root/nic/nginxplus-ingress-nap-dos
      tag: "2.2.2"
  
    ## Support for App Protect
    appprotect:
      enable: true
  
    ## Support for App Protect Dos
    appprotectdos:
      enable: true
  
    ingressClass: nginx
  
    ## Enable the custom resources.
    enableCustomResources: true
  
    ## Enable preview policies. This parameter is deprecated. To enable OIDC Policies please use controller.enableOIDC instead.
    enablePreviewPolicies: false
  
    ## Enable OIDC policies.
    enableOIDC: true
  
    globalConfiguration:
      ## Creates the GlobalConfiguration custom resource. Requires controller.enableCustomResources.
      create: true
  
      ## The spec of the GlobalConfiguration for defining the global configuration parameters of the Ingress Controller.
      spec: {}
        # listeners:
        # - name: dns-udp
        #   port: 5353
        #   protocol: UDP
        # - name: dns-tcp
        #   port: 5353
        #   protocol: TCP
  
    ## Enable custom NGINX configuration snippets in Ingress, VirtualServer, VirtualServerRoute and TransportServer resources.
    enableSnippets: true
  
    service:
      ## Creates a service to expose the Ingress Controller pods.
      create: true
      ## The type of service to create for the Ingress Controller.
      type: NodePort
  
    ## Enable collection of latency metrics for upstreams. Requires prometheus.create.
    enableLatencyMetrics: true
  
  prometheus:
    ## Expose NGINX or NGINX Plus metrics in the Prometheus format.
    create: true
  
    ## Configures the port to scrape the metrics.
    port: 9113
  
    ## Specifies the namespace/name of a Kubernetes TLS Secret which will be used to protect the Prometheus endpoint.
    secret: ""
  
    ## Configures the HTTP scheme used.
    scheme: http

- 1-3行目でNSMとの接続を有効にしています
- 6-9行目でNGINX Plusを有効にし、先程作成したImageを指定しています
- 12-13行目でNAP WAFを、16-17行目でNAP DoSを有効にしています
- 19行目でIngress Classとして ``nginx`` を指定しています
- 56-67行目でPrometheusに必要なパラメータを指定しています

続けて ``nic2`` で指定するパラメータの内容を確認します。
nic1 との差分を中心に確認します

.. code-block:: cmdin

  cat ~/f5j-nginx-observability-lab/prep/helm/nic2-addvalue.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1-3, 19

  nginxServiceMesh:
    enable: false
    enableEgress: false
  
  controller:
    nginxplus: true
    image:
      repository: registry.example.com/root/nic/nginxplus-ingress-nap-dos
      tag: "2.2.2"
  
    ## Support for App Protect
    appprotect:
      enable: true
  
    ## Support for App Protect Dos
    appprotectdos:
      enable: true
  
    ingressClass: nginx2
  
    ## Enable the custom resources.
    enableCustomResources: true
  
    ## Enable preview policies. This parameter is deprecated. To enable OIDC Policies please use controller.enableOIDC instead.
    enablePreviewPolicies: false
  
    ## Enable OIDC policies.
    enableOIDC: true
  
    globalConfiguration:
      ## Creates the GlobalConfiguration custom resource. Requires controller.enableCustomResources.
      create: true
  
      ## The spec of the GlobalConfiguration for defining the global configuration parameters of the Ingress Controller.
      spec: {}
        # listeners:
        # - name: dns-udp
        #   port: 5353
        #   protocol: UDP
        # - name: dns-tcp
        #   port: 5353
        #   protocol: TCP
  
    ## Enable custom NGINX configuration snippets in Ingress, VirtualServer, VirtualServerRoute and TransportServer resources.
    enableSnippets: true
  
    service:
      ## Creates a service to expose the Ingress Controller pods.
      create: true
      ## The type of service to create for the Ingress Controller.
      type: NodePort
  
    ## Enable collection of latency metrics for upstreams. Requires prometheus.create.
    enableLatencyMetrics: true
  
  prometheus:
    ## Expose NGINX or NGINX Plus metrics in the Prometheus format.
    create: true
  
    ## Configures the port to scrape the metrics.
    port: 9113
  
    ## Specifies the namespace/name of a Kubernetes TLS Secret which will be used to protect the Prometheus endpoint.
    secret: ""
  
    ## Configures the HTTP scheme used.
    scheme: http

- NSMとの接続を利用しないため、1-3行目の設定を無効(false)にしています
- 19行目でIngress Classとして `nginx2` を指定しています。 (nic1はnginx)

NICをそれぞれデプロイします

.. code-block:: cmdin
  
  cd ~/kubernetes-ingress/deployments/helm-chart
  cp ~/f5j-nginx-observability-lab/prep/helm/nic1-addvalue.yaml .
  cp ~/f5j-nginx-observability-lab/prep/helm/nic2-addvalue.yaml .
  helm upgrade --install nic1 -f nic1-addvalue.yaml . -n nginx-ingress
  helm upgrade --install nic2 -f nic2-addvalue.yaml . -n nginx-ingress

デプロイした結果を確認します

.. code-block:: cmdin
  
  helm list -n nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                                     APP VERSION
  appdos-arbitrator       nginx-ingress   1               2022-06-28 22:45:48.828679008 +0000 UTC deployed        nginx-appprotect-dos-arbitrator-0.1.0     1.1.0
  nic1                    nginx-ingress   1               2022-06-30 07:43:18.437887299 +0000 UTC deployed        nginx-ingress-0.13.2                      2.2.2
  nic2                    nginx-ingress   1               2022-06-30 07:44:26.615809096 +0000 UTC deployed 

Podが正しく作成されていることを確認します

.. code-block:: cmdin
  
  kubectl get pod -n nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  
  NAME                                                              READY   STATUS    RESTARTS      AGE
  appdos-arbitrator-nginx-appprotect-dos-arbitrator-844bdf64qjw9l   1/1     Running   1 (25h ago)   32h
  nic1-nginx-ingress-69d574d9fb-lnv9f                               1/1     Running   0             81s
  nic2-nginx-ingress-857cf9d78d-vzh9w                               1/1     Running   0             12s

NICへ通信を転送するための設定を行います。

NodePortの情報を確認します。

.. code-block:: cmdin
  
  kubectl get svc -n nginx-ingress | grep nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  
  nic1-nginx-ingress       NodePort    10.104.228.200   <none>        80:31430/TCP,443:32486/TCP   154m
  nic2-nginx-ingress       NodePort    10.106.138.240   <none>        80:30730/TCP,443:31903/TCP   152m

それぞれに表示されているポート番号を確認してください。これらの情報を元に、NGINXの設定を作成します。

.. code-block:: cmdin
  
  vi ~/f5j-nsm-lab/prep/nginx.conf

以下の内容を参考に、先程確認したNodePortで割り当てられたポート番号宛に通信を転送するように、NGINXを設定します。

.. code-block:: yaml
  :linenos:
  :caption: nginx.conf
  :emphasize-lines: 7,11,18,22
  
  # TCP/UDP load balancing
  #
  stream {
      ##  TCP/UDP LB for NIC/NSM ingressclass
      server {
          listen 80;
          proxy_pass localhost:31430;  # nic1 http port of NodePort
      }
      server {
          listen 443;
          proxy_pass localhost:32486;  # nic 1 https port of NodePort
      }
  
  
      ##  TCP/UDP LB for NIC2 nginx2 ingressclass
      server {
          listen 8080;
          proxy_pass localhost:30730;  # nic2 http port of NodePort
      }
      server {
          listen 8443;
          proxy_pass localhost:31903;  # nic2 https port of NodePort
      }
  
  }

設定をコピーし、反映します

.. code-block:: cmdin
  
  sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf-
  sudo cp ~/f5j-nsm-lab/prep/nginx.conf /etc/nginx/nginx.conf
  sudo nginx -s reload

