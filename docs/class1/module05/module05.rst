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

 .. image:: ./media/grafana-explore-logql1.jpg
    :width: 600

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

 .. image:: ./media/grafana-explore-logql2.jpg
    :width: 600

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

 .. image:: ./media/grafana-explore-logql3.jpg
    :width: 400

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
