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
  
  while : ; do echo "Demo Traffic Start" ; \
  bash ~/f5j-nginx-observability-lab/demotraffic/attack-to-bookinfo.sh ; \
  bash ~/f5j-nginx-observability-lab/demotraffic/dummy-access-to-bookinfo.sh ; \
  bash ~/f5j-nginx-observability-lab/demotraffic/other-traffic.sh ; \
  done ;

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

3. ステータスの確認
====

1. NIC Dashboard
----

2. NSM Dashboard
----

3. Loki Dashboard
----

4. Jaeger の確認
----
