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
