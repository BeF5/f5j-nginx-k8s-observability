アプリケーションの外部公開・必要な設定のデプロイ
####

1. NGINX Ingress Controllerの設定
====

外部からアプリケーションに接続するため、NGINX Ingress Controllerを設定します。
また、今回はNIC、NAP WAFのログをGrafana Lokiで管理するため、すでに設定したLokiの設定に合わせたフォーマットでログを出力するよう設定します。

.. code-block:: cmdin

  cd ~/f5j-nginx-observability-lab/prep/nic
  # for access observability tools
  kubectl apply -f monitor-jaeger-vs.yaml
  kubectl apply -f monitor-loki-grafana-vs.yaml
  kubectl apply -f monitor-prometheus-vs.yaml
  
  # for changing NIC custom log format
  kubectl apply -f nic1-custom_log_format.yaml
  kubectl apply -f nic2-custom_log_format.yaml


デプロイした結果を確認します

.. code-block:: cmdin

  kubectl get vs -A

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAMESPACE   NAME              STATE   HOST                   IP    PORTS   AGE
  monitor     loki-grafana-vs   Valid   grafana.example.com                  26s
  monitor     jaeger-vs         Valid   jaeger.example.com                   32s
  monitor     prometheus-vs     Valid   prometheus.example.com               40s


踏み台ホストのブラウザ(Chrome)よりそれぞれのサービスのURLへアクセスいただくことにより、画面をご覧頂くことが可能です。


- Prometheus: `http://prometheus.example.com:8080/ <http://prometheus.example.com:8080/>`__

   .. image:: ./media/prometheus-top.jpg
      :width: 400

- Jaeger: `http://jaeger.example.com:8080/ <http://jaeger.example.com:8080/>`__

   .. image:: ./media/jaeger-top.jpg
      :width: 400

- Grafana: `http://grafana.example.com:8080/ <http://grafana.example.com:8080/>`__

   .. image:: ./media/grafana-login.jpg
      :width: 400

2. Grafana Datasouce の追加
====

踏み台サーバのデスクトップのショートカットから ``Chrome`` を実行し、以下のURLにアクセスします

- `http://grafana.example.com:8080/ <http://grafana.example.com:8080/>`__

ログイン画面が表示されます。なお、インストールするバージョンによりビジュアルが異なる場合があります。

   .. image:: ./media/grafana-login.jpg
      :width: 400

Grafanaにログインするためにパスワードの情報を取得します。

.. code-block:: cmdin
  
  kubectl get secret --namespace monitor loki-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  jFQSgKatKfJQ816K81qkPYIB2v6FvYjyAPE5mnpt


ユーザ名 ``admin`` 、そして確認したパスワードを入力しログインしてください

   .. image:: ./media/grafana-login2.jpg
      :width: 400

画面左メニューの ``Configuration (歯車のマーク)`` > ``DataSource`` を開き、 ``Add data source`` をクリックしてください

   .. image:: ./media/grafana-add-datasource.jpg
      :width: 400

DataSourceにPrometheusの追加をします

   .. image:: ./media/grafana-add-prometheus.jpg
      :width: 400

URL に `http://prometheus-server <http://prometheus-server>`__ と入力し、 ``Save & test`` をクリックしてください

   .. image:: ./media/grafana-add-prometheus2.jpg
      :width: 400

DataSourceにJaegerの追加をします

   .. image:: ./media/grafana-add-jaeger.jpg
      :width: 400

URL に `http://jaeger-query:16686 <http://jaeger-query:16686>`__ と入力し、 
``Filter by Trace ID`` 、 ``Filter by Span ID`` 、 ``Enable Node Graph`` を有効にしてください。
その後、 ``Save & test`` をクリックしてください。

   .. image:: ./media/grafana-add-jaeger2.jpg
      :width: 400

Lokiはデプロイ時点で設定されています。以下のような結果になることを確認してください

   .. image:: ./media/grafana-datasource-list.jpg
      :width: 400

   .. image:: ./media/grafana-loki.jpg
      :width: 400

3. サンプルアプリケーションのデプロイ
====

サンプルアプリケーションに必要となる、NGINX Ingress Controllerを設定します。

.. code-block:: cmdin
  
  # for access bookinfo application with NAP WAF / custom log format
  kubectl apply -f simple-ap.yaml -n staging
  kubectl apply -f ap-logconf.yaml -n staging
  kubectl apply -f waf.yaml -n staging
  kubectl apply -f staging-bookinfo-nap-vs.yaml

デプロイした結果を確認します

.. code-block:: cmdin

  kubectl get vs -A

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAMESPACE   NAME              STATE   HOST                   IP    PORTS   AGE
  monitor     loki-grafana-vs   Valid   grafana.example.com                  26s
  monitor     jaeger-vs         Valid   jaeger.example.com                   32s
  monitor     prometheus-vs     Valid   prometheus.example.com               40s
  staging     bookinfo-vs       Valid   bookinfo.example.com                 96s
  
.. code-block:: cmdin

  kubectl get aplogconf,appolicy,policy -n staging

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                                  AGE
  aplogconf.appprotect.f5.com/logconf   83s
  
  NAME                                   AGE
  appolicy.appprotect.f5.com/simple-ap   87s
  
  NAME                              STATE   AGE
  policy.k8s.nginx.org/waf-policy   Valid   80s

NSM Labで利用した bookinfo のアプリケーションをデプロイします。
詳細は `NSM サンプルアプリケーションのデプロイ <https://f5j-nginx-service-mesh.readthedocs.io/en/latest/class1/module03/module03.html#id1>`__ を参照してください

.. code-block:: cmdin
  
  kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/bookinfo/platform/kube/bookinfo.yaml -n staging 

.. code-block:: cmdin
  
  kubectl get pod -n staging

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                              READY   STATUS    RESTARTS   AGE
  details-v1-7f4669bdd9-87hp5       2/2     Running   0          2m21s
  productpage-v1-5586c4d4ff-mjsr9   2/2     Running   0          2m20s
  ratings-v1-6cf6bc7c85-zzbsc       2/2     Running   0          2m21s
  reviews-v1-7598cc9867-djmm8       2/2     Running   0          2m21s
  reviews-v2-6bdd859457-gt6wb       2/2     Running   0          2m21s
  reviews-v3-6c98f9d7d7-f8jk8       2/2     Running   0          2m21s

