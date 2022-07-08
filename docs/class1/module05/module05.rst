ステータスの確認
####

1. Dashboardのデプロイ
====

Grafana 左側のメニュー ``＋`` にマウスカーソルを合わせ、 ``Import`` をクリックし、DashboardのImport画面を開いてください

   .. image:: ./media/grafana-import-dashboard.jpg
      :width: 400

以下の画面が表示されますので、各DashboardをImportしてください

   .. image:: ./media/grafana-import-dashboard2.jpg
      :width: 400


1. NIC Dashboard
----

Import画面を開き、以下のURLの内容をコピーし ``Import via panel json`` に貼り付け、 ``Load`` をクリックしてください

- `GitHub NIC-Dashboard.json <https://raw.githubusercontent.com/BeF5/f5j-nginx-observability-lab/master/dashboard/NIC-Dashboard.json>`__

Jsonの読み込みに成功すると、以下の画面が表示されるので ``Prometheus`` の欄にGrafanaのDatasourceから ``Prometheus`` を選択し、 ``Import`` をクリックしてください

   .. image:: ./media/grafana-import-nic-dashboard.jpg
      :width: 400

2. NSM Dashboard
----

Import画面を開き、以下のURLの内容をコピーし ``Import via panel json`` に貼り付け、 ``Load`` をクリックしてください

- `GitHub NSM-Dashboard.json <https://raw.githubusercontent.com/BeF5/f5j-nginx-observability-lab/master/dashboard/NSM-Dashboard.json>`__

Jsonの読み込みに成功すると、以下の画面が表示されるので ``Prometheus`` の欄にGrafanaのDatasourceから ``Prometheus`` を選択し、 ``Import`` をクリックしてください

   .. image:: ./media/grafana-import-nsm-dashboard.jpg
      :width: 400

3. Loki Dashboard
----

Import画面を開き、以下のURLの内容をコピーし ``Import via panel json`` に貼り付け、 ``Load`` をクリックしてください

- `GitHub Loki-Dashboard.json <https://raw.githubusercontent.com/BeF5/f5j-nginx-observability-lab/master/dashboard/Loki-Dashboard.json>`__

Jsonの読み込みに成功すると、以下の画面が表示されるので ``Loki`` の欄にGrafanaのDatasourceから ``Loki`` を選択し、 ``Import`` をクリックしてください

   .. image:: ./media/grafana-import-loki-dashboard.jpg
      :width: 400

2. デモトラフィックの実行
====

以下コマンドを実行し、デモトラフィックを実行してください

.. code-block:: cmdin
  
  while : ; do echo "Demo Traffic Start!!" ; \
  bash ~/f5j-nginx-observability-lab/demotraffic/attack-to-bookinfo.sh ; \
  bash ~/f5j-nginx-observability-lab/demotraffic/dummy-access-to-bookinfo.sh ; \
  bash ~/f5j-nginx-observability-lab/demotraffic/other-traffic.sh ; \
  done ;

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Demo Traffic Start!!
  Thu Jul  7 12:11:53 UTC 2022
  attack to bookinfo start
  done
  Thu Jul  7 12:12:13 UTC 2022
  dummy access to bookinfo start
  done
  Thu Jul  7 12:12:33 UTC 2022
  other traffic start
  ... (以下ループ)

3. ステータスの確認
====

.. NOTE::
   ダッシュボードに結果が反映するまで時間がかかる場合があります
   正しく読み込まれない場合などは、対象のダッシュボードを再度開き、状態を確認してください

  

1. NIC Dashboard
----

画面左上 ``NIC`` から対象のNICを選択できます。情報を確認したい ``NIC`` を選択してください

 .. image:: ./media/grafana-nic-selection.jpg
    :width: 400

``Success Rates Over Time`` のグラフを参考に詳細を確認します

 .. image:: ./media/grafana-nic-successrate-menu.jpg
    :width: 400

- ``View`` : 対象の項目を画面全体で確認できます

 .. image:: ./media/grafana-nic-successrate-view.jpg
    :width: 400

- ``Edit`` : 表示内容の条件など詳細を確認、変更することができます。変更内用を表示に反映する場合には右上の ``Apply`` 、 Dashboardに反映する場合には ``Save`` をクリックしてください。変更を破棄する場合には ``Discard`` をクリックしてください。

 .. image:: ./media/grafana-nic-successrate-edit.jpg
    :width: 400

- ``Explore`` : 表示の詳細を確認できます

 .. image:: ./media/grafana-nic-successrate-explore.jpg
    :width: 400

- ``Inspect`` > ``Data`` : 表示内容の値を確認できます

 .. image:: ./media/grafana-nic-successrate-inspect_data.jpg
    :width: 400


2. NSM Dashboard
----
 .. image:: ./media/grafana-nsm-dashboard-top.jpg
    :width: 400

3. Loki Dashboard
----

 .. image:: ./media/grafana-loki-dashboard-top.jpg
    :width: 400

4. Jaeger の確認
----

 .. image:: ./media/grafana-explore-jaeger.jpg
    :width: 400

 .. image:: ./media/grafana-explore-jaeger2.jpg
    :width: 400

 .. image:: ./media/grafana-explore-jaeger3.jpg
    :width: 400

Tips1. 各種ステータスの確認
====

Grafana の Explore より、各データソースの詳細を確認することが可能です

 .. image:: ./media/grafana-explore.jpg
    :width: 400

Prometheus ステータスの確認
----

Metric Browser をクリックし、Prometheusが取得したMetricsを確認できます

 .. image:: ./media/grafana-explore-prometheus.jpg
    :width: 400

 .. image:: ./media/grafana-explore-prometheus2.jpg
    :width: 400

PrometheusはPromQLという書式で様々にMetricsを操作し、情報を出力することが可能です。
PromQLについては以下のドキュメントを参照してください。

- `Prometheus Querying Functions <https://prometheus.io/docs/prometheus/latest/querying/functions/>`__
- `Prometheus Querying Examples <https://prometheus.io/docs/prometheus/latest/querying/examples/>`__

 .. image:: ./media/grafana-explore-prometheus-promql.jpg
    :width: 400

 .. image:: ./media/grafana-explore-prometheus-promql2.jpg
    :width: 400

Loki ステータスの確認
----

 .. image:: ./media/grafana-explore-loki.jpg
    :width: 400

 .. image:: ./media/grafana-explore-loki2.jpg
    :width: 400

 .. image:: ./media/grafana-explore-loki3.jpg
    :width: 400

 .. image:: ./media/grafana-explore-loki4.jpg
    :width: 400

LokiはLogQLという書式で様々にMetricsを操作し、情報を出力することが可能です。
LogQLについては以下のドキュメントを参照してください。

- `Grafana Log queries <https://grafana.com/docs/loki/latest/logql/log_queries/>`__

 .. image:: ./media/loki-logql.jpg
    :width: 400

Log browser 右側にLogQLで記述した条件を入力し、 ``Ctrl + Enter`` または 画面右上の ``Run query`` をクリックすると結果が表示されます


出力例1: logtype securitylog のログで、Bot機能で curl と判定されたものを抽出
^^^^

.. code-block:: cmdin
  :caption: Log Query文字列
  
  {namespace="nginx-ingress"} | json | logtype="securitylog" | bot_signature_name="curl"


+------------------------------+------------------------------------------------------+
|{namespace="nginx-ingress"}   | 対象のログを示すラベルの指定                         |
+------------------------------+------------------------------------------------------+
| \| json                      | ``json`` 形式でログデータをパース                    |
+------------------------------+------------------------------------------------------+
| \| logtype="securitylog"     | ``logtype`` が ``securitylog`` のログをフィルタ      |
+------------------------------+------------------------------------------------------+
| \| bot_signature_name="curl" | ``bot_signature_name`` が ``curl`` のログをフィルタ  |
+------------------------------+------------------------------------------------------+

 .. image:: ./media/grafana-explore-logql1-graph.jpg
    :width: 400


出力例2: logtype accessylog のログで、grafana.example.com 宛の接続を抽出
^^^^


.. code-block:: cmdin
  :caption: Log Query文字列
  
  {namespace="nginx-ingress"} | json | logtype="accesslog" | server_name="grafana.example.com"



+--------------------------------------+--------------------------------------------------------------+
|{namespace="nginx-ingress"}           | 対象のログを示すラベルの指定                                 |
+--------------------------------------+--------------------------------------------------------------+
| \| json                              | ``json`` 形式でログデータをパース                            |
+--------------------------------------+--------------------------------------------------------------+
| \| logtype="accesslog"               | ``logtype`` が ``accesslog`` のログをフィルタ                |
+--------------------------------------+--------------------------------------------------------------+
| \| server_name="grafana.example.com" | ``server_name`` が ``grafana.example.com`` のログをフィルタ  |
+--------------------------------------+--------------------------------------------------------------+

 .. image:: ./media/grafana-explore-logql2-graph.jpg
    :width: 400

出力例3: logtype accessylog のログで、5分毎の集計結果を、server_name 毎に集計して表示
^^^^

.. code-block:: cmdin
  :caption: Log Query文字列
  
  sum by (server_name) (count_over_time(
  {namespace="nginx-ingress"} | json | logtype="accesslog" 
  [5m]))


+--------------------------------------+--------------------------------------------------------------+
|sum by (server_name)(                 | 抽出した条件の結果を合計し、指定のパラメータごとに表示       |
+--------------------------------------+--------------------------------------------------------------+
| count_over_time(                     | 指定した期間のログをカウント                                 |
+--------------------------------------+--------------------------------------------------------------+
|  {namespace="nginx-ingress"}         | 対象のログを示すラベルの指定                                 |
+--------------------------------------+--------------------------------------------------------------+
|  \| json                             | ``json`` 形式でログデータをパース                            |
+--------------------------------------+--------------------------------------------------------------+
|  \| logtype="accesslog"              | ``logtype`` が ``accesslog`` のログをフィルタ                |
+--------------------------------------+--------------------------------------------------------------+
|[5m]))                                | ``count_over_time`` の期間を指定 (5m=5分)                    |
+--------------------------------------+--------------------------------------------------------------+

 .. image:: ./media/grafana-explore-logql3-graph.jpg
    :width: 400

Loki Promtail 設定
----

Lokiで利用するPromtailの設定について紹介します。

ログデータに対し適切な処理を行うことで、Lokiでのログの閲覧がより容易になります。

Helm Chartの `values.yaml config <https://github.com/grafana/helm-charts/blob/main/charts/promtail/values.yaml#L237>`__ に指定された情報が、
Template の `secret.yaml <https://github.com/grafana/helm-charts/blob/main/charts/promtail/templates/secret.yaml>`__ に読み込まれ、
Promtail の設定として反映されます。

記述内容の詳細については、以下のドキュメントを参照して下さい。

- `Grafana Configuring Promtail <https://grafana.com/docs/loki/latest/clients/promtail/configuration/>`__

ラボの設定ファイルを元に開設します。設定ファイル全体の構成は以下のとおりです。

 .. image:: ./media/promtail-config.jpg
    :width: 400

この設定項目の中で ``scrape_configs`` がLogデータの取得、及び整形・データの抽出を記述する箇所となります。
``scrape_configs`` を確認します

 .. image:: ./media/promtail-config-scrape_configs.jpg
    :width: 400

それぞれ、 ``loki-scrape-addvalue.yaml`` 、 ``loki-scrape.yaml`` で指定した内容が設定ファイルに反映されています。

Syslogサーバの参考設定を示す ``job_name: syslog`` はSyslogで受けたLogデータを処理する設定となります。
各Podのログの取得は ``job_name: kubernetes-pods`` に記述しています。

``scrape_configs`` では大きく分けて以下の設定を記述します

- Scraping (Service Discovery)：

  - ログデータを取得します。
  - Scrapeの処理は ``Kubernetes`` 、 ``Windows Event`` 、 ``Journal(Linux)`` 、 ``syslog`` などがあります
  - Scrapeの内容に応じた情報を持つラベルがあり、それらを元にログの分類やタグ付けが可能です。その処理を ``Relabel`` で記述します
  - 詳細は、設定ファイルの記述方法や `Grafana Configuring Promtail <https://grafana.com/docs/loki/latest/clients/promtail/configuration/>`__ 、 `Grafana Promtail Scraping <https://grafana.com/docs/loki/latest/clients/promtail/scraping/>`__ を確認してください

- Relabel : 

  - ログデータを条件に従ってラベルの付与（付け替え など）を行います
  - Scrapeに応じたラベルが利用可能です。ラベルは予め予約された文字列が利用され、 ``アンダースコア2つ (__)`` から始まる文字列で指定されます
  - 詳細は、設定ファイルの記述方法や `Grafana Configuring Promtail <https://grafana.com/docs/loki/latest/clients/promtail/configuration/>`__ 、 `Grafana Promtail Scraping <https://grafana.com/docs/loki/latest/clients/promtail/scraping/>`__ を確認してください

- Pipeline : 

  - ログデータを各種情報に合わせて、検索、整形、変更、ラベルなどを行います。Pipelineの処理はStageという構成で呼ばれ、記述に合わせた処理を行います
  - Pipeline の詳細は `Grafana Promtail <https://grafana.com/docs/loki/latest/clients/promtail/pipelines/>`__ を参照してください
  - Stage の詳細、及び関数の詳細は `Grafana Promtail Stages <https://grafana.com/docs/loki/latest/clients/promtail/stages/>`__ を参照してください

``Pipeline`` ではログデータに関する処理を記述します。
Stageは ``Parsing`` 、 ``Transoform`` 、 ``Action`` 、 ``Filtering`` の4種類に分類されており、以下のような内容となります。

- Parsing stages: データの構造を指定の内容でパースします

  - docker: ログデータをDocker Formatでパースします
  - cri: ログデータをCRI Formatでパースします
  - json: ログデータをJson Formatでパースします

- Transform stages: ログラインの構成を変更・変形します

  - template: GoのTemplateを使って、データを処理します
  
- Action stages: ログに関連するデータをしていします

  - timestamp: ログエントリの時刻情報を指定します

- Filtering stages: 対象とするログの選択や、ログの転送に関する設定をします

  - match: 指定した条件に該当するログに対してstageを実行します 

| これらを組み合わせ意図した形式のログデータをLokiで扱えるようにします。
| ラボの設定では以下のような意図の設定を記述しています。

 .. image:: ./media/promtail-config-scrape_configs2.jpg
    :width: 400

各Stage、関数の記述イメージは以下です。

 .. image:: ./media/promtail-config-scrape_configs3.jpg
    :width: 400

これらの処理によりLokiでのデータの扱いが以下の様に容易になります

- NICのAccess Log(logtype accesslog)、NAP WAFのLog(logtype accesslog)をjsonで容易に扱える様に変更
- NAP WAFのLogでJSONパースでエラーとなる文字列の置換、及び該当データがない場合の文字列をAccess Logと統一

LogQLは柔軟な記述が可能となりますので、ダッシュボードの記述内容も合わせてご確認ください

Tips2. ラボが正しく動作しない場合
====

- 対象のリソースを削除し、再度作成する

  - helm でデプロイしたリソースの削除

  .. code-block:: cmdin
    
    helm uninstall <resouce name>  <-n namespace>
  

  - kubectl でデプロイしたリソースの削除

  .. code-block:: cmdin
    
    kubectl delete <resource type> <resouce name> <-n namespace>
    kubectl delete -f <yaml file> <-n namespace>


- 各リソースへの疎通を確認する

  - デモアプリケーションへの疎通を確認する

  .. code-block:: cmdin
    
    curl -v -H "Host: bookinfo.example.com" "http://127.0.0.1/productpage"  | grep "<title>"


  - 各監視ツールへの疎通を確認する

  .. code-block:: cmdin
    
    curl -v -H "Host: grafana.example.com" "http://127.0.0.1:8080/login" | grep "<title>"
    curl -v -H "Host: prometheus.example.com" "http://127.0.0.1:8080/graph" | grep "<title>"
    curl -v -H "Host: jaeger.example.com" "http://127.0.0.1:8080/" | grep "<title>"


  - WAFでブロックされることを確認する

  .. code-block:: cmdin
    
    curl -v -H "Host: bookinfo.example.com" "http://127.0.0.1/productpage?a=<script>" 
  