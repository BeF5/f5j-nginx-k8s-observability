アプリケーションの外部公開・必要な設定のデプロイ
####

1. NGINX Ingress Controllerの設定
====

外部からアプリケーションに接続するため、NGINX Ingress Controllerを設定します。
また、今回はNIC、NAP WAFのログをGrafana Lokiで管理するため、すでに設定したLokiの設定に合わせたフォーマットでログを出力するよう設定します。

.. code-block:: cmdin

  ## cd ~/observability/prep/
  
  # for access observability tools
  monitor-jaeger-vs.yaml
  monitor-loki-grafana-vs.yaml
  monitor-prometheus-vs.yaml
  
  # for changing NIC custom log format
  nic1-custom_log_format.yaml
  nic2-custom_log_format.yaml
  
  # for access bookinfo application with NAP WAF / custom log format
  simple-ap.yaml
  ap-logconf.yaml
  waf.yaml
  staging-bookinfo-nap-vs.yaml

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

踏み台ホストのブラウザ(Chrome)よりそれぞれのサービスのURLへアクセスいただくことにより、画面をご覧頂くことが可能です。


- Prometheus: ``http://prometheus.example.com:8080/``

   .. image:: ./media/prometheus-top.jpg
      :width: 400

- Jaeger: ``http://jaeger.example.com:8080/``

   .. image:: ./media/jaeger-top.jpg
      :width: 400

- Grafana: ``http://grafana.example.com:8080/``

   .. image:: ./media/grafana-top.jpg
      :width: 400

2. Grafana Datasouce の追加


3. サンプルアプリケーションのデプロイ
====

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

