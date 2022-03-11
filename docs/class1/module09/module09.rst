F5 DCS API Security (作成中)
####

F5 DCS で API Security を利用する方法や、各種設定について紹介します

マニュアルは以下のページを参照してください
- `API Endpoint - Discovery & Control <https://docs.cloud.f5.com/docs/how-to/app-security/apiep-discovery-control>`__


1. F5 DCS API Security について
====

F5 DCS API Securityは、以下の特徴を備えております

- APIスキーマとSwaggerファイルを生成し、APIエンドポイントの手動トラッキングを最小限に抑える
- 不審なリクエストをブロックし、データ漏洩を防止
- APIセキュリティポリシーの設定・導入にかかる時間を短縮
- ユーザの異常な行動を分析、特定し、悪意のある攻撃をブロック

F5 DCS WAAPはこれらの高度なセキュリティをアプリケーションの迅速な展開に合わせて自由にご利用いただける環境を実現します

2. API Discovery の設定
====

F5 DCS WAAPではバックエンドアプリケーションで利用されているAPIの情報を学習し、その結果をダッシュボードで確認いただくことが可能です。
この機能により、バックエンドアプリケーションの提供するAPIのEndopint、Method、通信状況を効率的に把握できます

.. NOTE::
    APIのDiscovery機能の結果が画面に表示されるまで、数時間以上のかなりの時間を要する場合があります。
    継続的に発生するトラフィックを対象としており、APIへの通信頻度が少ない場合、正しく結果が表示されない場合があります。
    本設定の後定常的にトラフィックが発生する状態を長時間維持し、その後、結果の確認ができるような環境での動作確認をおすすめします。

API Discoveryでは以下用語を用いる場合があります

======================================= ============================================================
EP (End Point)                          API Gateway の End Point。対象とするURL PathとMethodを示します
--------------------------------------- ------------------------------------------------------------
PDF (Probability Distribution Function) | API EPの通信に関するメトリック情報。以下の内容を取得する
                                        | ・ Request / Response size
                                        | ・ データあり / なし 時の遅延時間
                                        | ・ Request rate
                                        | ・ Error rate
                                        | ・ Response スループット
======================================= ============================================================

1. API Discovery の設定
----

作成済みのHTTP Load Balancerに Malicious User Detaction & Mitigation に関連するパラメータを設定します。
HTTP Load Balancer の設定手順は `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module03/module03.html>`__ を参照ください

画面左側 Manage欄の ``Load Balancers`` 、 ``HTTP Load Balancers`` を開き、対象のLoad Balancerを表示し画面右側に遷移します。

   .. image:: ../module05/media/dcs-edit-lb.jpg
       :width: 400

すでに作成済みのオブジェクトを変更する場合、対象のオブジェクト一番右側 ``‥`` から、 ``Manage Configuration`` をクリックします

   .. image:: ../module05/media/dcs-edit-lb2.jpg
       :width: 400

設定の結果が一覧で表示されます。画面右上 ``Edit Configuration`` から設定の変更します。
Security Configuration 欄 右上の ``Show Advanced Fields`` をクリックします。

API Discovery を設定します。
今回は、単一のLoad Balancerを対象とした設定となりますので、 ``ML Config`` で ``Single Load Balancer Application`` を選択します。
その配下に表示される ``API Discovery`` で ``Enable API Discovery`` を選択してください。
その他機能は利用しませんので、 ``無効 (Disable)`` を選択してください

   .. image:: ./media/dcs-edit-lb-api-discovery.jpg
       :width: 400

1. サンプルリクエストの送付
----

Curlコマンドによりサンプルリクエストを送付します。
この例ではいずれのリクエストについても同等のJSONを応答するサーバに対してリクエストを送ります。サンプルリクエストは以下の内容です。

.. code-block:: bash
  :linenos:
  :caption: Curl コマンドを使った https://echoapp.f5demo.net へのサンプルリクエスト

  $ curl -X GET -vks https://echoapp.f5demo.net/ ;
  
  ** 省略 **

  > GET / HTTP/2
  > Host: echoapp.f5demo.net
  > User-Agent: curl/7.58.0
  
  ** 省略 **

  < HTTP/2 200
  < content-type: application/json
  < content-length: 670
  
  ** 省略 **

  {"request":{"headers":[["host","app2.test10demo.xyz"],["user-agent","curl/7.58.0"],["accept","*/*"],["x-forwarded-for","18.178.83.1"],["x-forwarded-proto","https"],["x-envoy-external-address","18.178.83.1"],["x-request-id","46aca87d-6141-4656-992f-4bde488f4c3d"],["content-length","0"]],"status":0,"httpversion":"1.1","method":"GET","scheme":"http","uri":"/","requestText":"","fullPath":"/"},"network":{"clientPort":"57613","clientAddress":"103.135.56.106","serverAddress":"192.168.16.2","serverPort":"80"},"ssl":{"isHttps":false},"session":{"requestId":"35f075bad07a58663f843875701a092e","connection":"2695","connectionNumber":"5"},"environment":{"hostname":"echoapp"}}

.. NOTE::
  API DiscoveryはOrigin ServerのAPIの仕様を担保するものではなく、あくまで通信の統計から判断出来るAPIの情報を表示する機能です。

.. code-block:: bash
  :linenos:
  :caption: API Disovery のための簡易なShell Scriptの作成

  $ cat << EOF > api-access.sh
  
  # GET
  curl -X GET -ks https://echoapp.f5demo.net/ ;
  curl -X GET -ks https://echoapp.f5demo.net/product ;
  curl -X GET -ks https://echoapp.f5demo.net/product/book ;
  curl -X GET -ks https://echoapp.f5demo.net/product/dvd ;
  curl -X GET -ks https://echoapp.f5demo.net/product/cd ;
  curl -X GET -ks https://echoapp.f5demo.net/product/game ;
  curl -X GET -ks https://echoapp.f5demo.net/product/stationery ;
  curl -X GET -ks https://echoapp.f5demo.net/rental ;
  curl -X GET -ks https://echoapp.f5demo.net/rental/book ;
  curl -X GET -ks https://echoapp.f5demo.net/rental/dvd ;
  curl -X GET -ks https://echoapp.f5demo.net/rental/cd ;
  curl -X GET -ks https://echoapp.f5demo.net/rental/game ;
  curl -X GET -ks https://echoapp.f5demo.net/rental/stationery ;
  curl -X GET -ks https://echoapp.f5demo.net/cart ;
  curl -X GET -ks https://echoapp.f5demo.net/top ;
  curl -X GET -ks https://echoapp.f5demo.net/img ;
  # POST
  curl -X POST -ks https://echoapp.f5demo.net/product/book -d '{ "id" : 1 , "title" : "dummy-book" }';
  curl -X POST -ks https://echoapp.f5demo.net/product/dvd  -d '{ "id" : 1 , "title" : "dummy-dvd" }';
  curl -X POST -ks https://echoapp.f5demo.net/product/cd   -d '{ "id" : 1 , "title" : "dummy-cd" }';
  curl -X POST -ks https://echoapp.f5demo.net/product/game -d '{ "id" : 1 , "title" : "dummy-game" }';
  curl -X POST -ks https://echoapp.f5demo.net/product/stationery -d '{ "id" : 1 , "title" : "dummy-stationery }';
  curl -X POST -ks https://echoapp.f5demo.net/rental/book -d '{ "id" : 1 , "title" : "dummy-book" }';
  curl -X POST -ks https://echoapp.f5demo.net/rental/dvd  -d '{ "id" : 1 , "title" : "dummy-dvd" }';
  curl -X POST -ks https://echoapp.f5demo.net/rental/cd   -d '{ "id" : 1 , "title" : "dummy-cd" }';
  curl -X POST -ks https://echoapp.f5demo.net/rental/game -d '{ "id" : 1 , "title" : "dummy-game" }';
  curl -X POST -ks https://echoapp.f5demo.net/rental/stationery -d '{ "id" : 1 , "title" : "dummy-stationery }';

  EOF

  # 適宜コマンドに実行権限を付与してください
  $ chomod +x api-access.sh

以下コマンドを実行します。20秒毎に先程作成したスクリプトよりリクエストを送信します。すべてのリクエストについて同一の応答が返ってきます
結果が表示されるまで、数時間以上要する場合があります。クライアントより長時間コマンドを実行してください。

.. code-block:: bash
  :linenos:
  :caption: API Discovery のためのサンプルリクエストの実行

  $ while : ; do sleep 20 ; date ; ./api-access.sh  ; done


一定時間、コマンドを実行してください。数時間放置の後、 ``Ctrl-C`` でコマンドを停止させてください

3. API Discovery の結果
----

次に画面左側、Meshの ``Service Mesh`` をクリックし、表示された項目の ``More`` をクリックします

   .. image:: ./media/dcs-mesh-api-discovery.jpg
       :width: 400

.. NOTE::
    対象のHTTP Load BalancerにLabelの割当がない場合、Namespace 名で項目が表示されます。Labelの割当がある場合、Labelが項目の名称として表示されます
    指定した期間にNamespaceやLabelなど複数のオブジェクトに対して通信がある場合、それらが項目として表示されます。

``API Endpoints`` のタブを開き、 ``Graph`` が選択され、結果が表示されていることを確認できます。
このGraphがAPI Discoveryによって把握できるAPIの情報となります。

URL Path を階層(Segment)毎に表示しており、各APIのEPが表示されます。

   .. image:: ./media/dcs-mesh-api-discovery2.jpg
       :width: 400

また、画面右上の ``Download Swagger`` より全体の構成を示すSwagger Fileをダウンロードすることが可能です。

各APIのEPの項目は以下を示します

========================= =========================================================================================================
Schema                    対象となるAPI End Pointの構成情報が表示可能であることを示します
------------------------- ---------------------------------------------------------------------------------------------------------
PDFs                      対象となるAPI End Pointのメトリクスの表示が可能であることを示します
------------------------- ---------------------------------------------------------------------------------------------------------
HTTP Method(サンプルはGET) 対象のURL PathのHTTP Methodを示します。同一Pathに複数のMethodが公開される場合それぞれ別の項目として表示されます
========================= =========================================================================================================

   .. image:: ./media/dcs-mesh-api-discovery3.jpg
       :width: 400

APIのEPにマウスオーバーするとポップアップで詳細が確認できます。

   .. image:: ./media/dcs-mesh-api-discovery4.jpg
       :width: 400

さらに、APIのEPをクリックするとそれらのメトリクスや構成情報を確認できます。

   .. image:: ./media/dcs-mesh-api-discovery5.jpg
       :width: 400

各メトリクスはマウスオーバーすると詳細が確認でき、クリックするとグラフで詳細を確認できます。

   .. image:: ./media/dcs-mesh-api-discovery6.jpg
       :width: 400

   .. image:: ./media/dcs-mesh-api-discovery7.jpg
       :width: 400

通信状況から把握した内容を構成情報として表示します。

   .. image:: ./media/dcs-mesh-api-discovery8.jpg
       :width: 400

Swagger タブを開くと、対象のAPI EPの構成情報をSwagger Fileとしてダウンロードすることができます。

   .. image:: ./media/dcs-mesh-api-discovery9.jpg
       :width: 400

画面上部の ``Table`` を選択すると、表敬式で情報を確認することができます。
各メトリクスは、 ``Graph`` で各API EPの情報を確認した時と同様の操作が可能です。

   .. image:: ./media/dcs-mesh-api-discovery10.jpg
       :width: 400

