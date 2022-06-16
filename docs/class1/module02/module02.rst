NGINX K8S 環境のセットアップ
####

1. NIC/NSMのセットアップ
====

NSMの手順に従って環境をセットアップします。
手順: `NSMのセットアップ <https://f5j-nginx-service-mesh.readthedocs.io/en/latest/class1/module02/module02.html>`__

NSMのInstallコマンドは以下の内容で実行してください。

.. NOTE::
  NSM v1.5.0 よりGrafana, Jaeger, Prometheus, Zipkinがインストールされませんので注意ください

.. code-block:: cmdin

  nginx-meshctl deploy --image-tag 1.4.0 --enabled-namespaces="prod,staging"  --mtls-mode=strict  --disable-auto-inject --nginx-lb-method round_robin
