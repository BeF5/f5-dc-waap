XC API Security 
####

※本資料の画面表示や名称は資料作成時点の画面表示を利用しております。アップデート等より表示が若干異なる場合がございます。

XC で API Security を利用する方法や、各種設定について紹介します

マニュアルは以下のページを参照してください

- `API Endpoint - Discovery & Control <https://docs.cloud.f5.com/docs/how-to/app-security/apiep-discovery-control>`__


1. XC API Security について
====

XC API Securityは、以下の特徴を備えております

- APIスキーマとSwaggerファイルを生成し、APIエンドポイントの手動トラッキングを最小限に抑える
- 不審なリクエストをブロックし、データ漏洩を防止
- APIセキュリティポリシーの設定・導入にかかる時間を短縮

XC WAAPはこれらの高度なセキュリティをアプリケーションの迅速な展開に合わせて自由にご利用いただける環境を実現します

2. API Discovery の設定
====

XC WAAPではバックエンドアプリケーションで利用されているAPIの情報を学習し、その結果をダッシュボードで確認いただくことが可能です。
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
HTTP Load Balancer の設定手順は `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module04/module04.html>`__ を参照ください

画面左側 Manage欄の ``Load Balancers`` 、 ``HTTP Load Balancers`` を開き、対象のLoad Balancerを表示し画面右側に遷移します。

   .. image:: ../module06/media/dcs-edit-lb.jpg
       :width: 400

すでに作成済みのオブジェクトを変更する場合、対象のオブジェクト一番右側 ``‥`` から、 ``Manage Configuration`` をクリックします

   .. image:: ../module06/media/dcs-edit-lb2.jpg
       :width: 400

設定の結果が一覧で表示されます。画面右上 ``Edit Configuration`` から設定の変更します。
Security Configuration 欄 右上の ``Show Advanced Fields`` をクリックします。

API Discovery を設定します。
今回は、単一のLoad Balancerを対象とした設定となりますので、 ``ML Config`` で ``Single Load Balancer Application`` を選択します。
その配下に表示される ``API Discovery`` で ``Enable API Discovery`` を選択してください。
その他機能は利用しませんので、 ``無効 (Disable)`` を選択してください

   .. image:: ./media/dcs-edit-lb-api-discovery.jpg
       :width: 400

2. サンプルリクエストの送付
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

Swagger Fileのサンプルは以下です

- :download:`API Discovery Sample Swagger File <./file/ves-io-http-loadbalancer-demo-echo-lb.json>`


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

Swagger Fileのサンプルは以下です

- :download:`特定API EP Sample Swagger File <./file/rental_book_GET.json>`


画面上部の ``Table`` を選択すると、表敬式で情報を確認することができます。
各メトリクスは、 ``Graph`` で各API EPの情報を確認した時と同様の操作が可能です。

   .. image:: ./media/dcs-mesh-api-discovery10.jpg
       :width: 400

4. API Document の確認
----

Swaggerが提供するSwagger EditorでダウンロードしたSwagger Fileがどのような形で表示されるか確認します。

`Swagger Editor <https://editor.swagger.io/>`__ を開いてください。

   .. image:: ./media/swagger-editor.jpg
       :width: 400

画面左側に、対象となる Swagger File の内容を貼り付けてください。
JSON形式の内容を貼り付ける場合、YAMLへの変換に関する確認が表示されますので ``OK`` をクリックしてください。

   .. image:: ./media/swagger-editor2.jpg
       :width: 400

貼り付けた結果より、右側に API Document が生成されていることが確認できます。
このように、Swagger Fileを利用することで、APIの構成を把握すると共に、APIの定義を公開する際にも有用であることが確認できます。

   .. image:: ./media/swagger-editor3.jpg
       :width: 400


3. Swagger File を利用した API Group による通信制御
====

Swagger Fileを用いてAPI Groupを定義します。
細かなAPIへの制御が可能です。

マニュアルは以下のページを参照してください

- `Import Swagger to Define and Control API Groups <https://docs.cloud.f5.com/docs/how-to/advanced-security/import-swagger-control-api-access>`__

1. Swagger File のImport
----

以下をSwagger Fileのサンプルとして紹介します。必要に応じてファイルをダウンロードしてください。

- :download:`User API Swagger File <./file/user-api.json>`
- :download:`REST API Swagger File <./file/rest-api.json>`

メニューより ``Web App & API Protection`` を開いてください。

   .. image:: ./media/dcs-console-waap.jpg
       :width: 400

画面左側 Manage欄の ``Files`` 、 ``Swagger Files`` を開き、 ``Add Swagger File`` をクリックしてください。

   .. image:: ./media/dcs-waap-add-swaggerfile.jpg
       :width: 400

``Name`` 欄に ``demo-app-user-api`` と入力し、 ``Upload File`` をクリックし、 ``User API Swagger File(user-api.json)`` をアップロードします。

   .. image:: ./media/dcs-waap-add-swaggerfile2.jpg
       :width: 400

REST API Swagger File に対し同様の手順を行います。
``Name`` 欄に ``demo-app-rest-api`` と入力し、 ``Upload File`` をクリックし、 ``REST API Swagger File(rest-api.json)`` をアップロードします。

   .. image:: ./media/dcs-waap-add-swaggerfile3.jpg
       :width: 400

   .. image:: ./media/dcs-waap-add-swaggerfile4.jpg
       :width: 400

Importが完了したSwagger FileのURL情報を取得します。 後の項目で利用しますので ``このURL情報をメモ`` しておいてください。

   .. image:: ./media/dcs-waap-get-swaggerurls.jpg
       :width: 400

このサンプルでは以下のような書式でURLが生成されます。

.. code-block:: bash
    https://f5-apac-ent.console.ves.volterra.io/api/object_store/namespaces/h-matsumoto/stored_objects/swagger/demo-app-user-api/v1-22-03-14


2. API Definition の作成
----

作成済みのHTTP Load Balancerに APIのAccess Control に関連するパラメータを設定します。
HTTP Load Balancer の設定手順は `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module03/module03.html>`__ を参照ください

本手順では、HTTP Load BalancerからAPI Definitionを定義します。

画面左側 Manage欄の ``Load Balancers`` 、 ``HTTP Load Balancers`` を開き、対象のLoad Balancerを表示し画面右側に遷移します。

   .. image:: ../module06/media/dcs-edit-lb.jpg
       :width: 400

すでに作成済みのオブジェクトを変更する場合、対象のオブジェクト一番右側 ``‥`` から、 ``Manage Configuration`` をクリックします

   .. image:: ../module06/media/dcs-edit-lb2.jpg
       :width: 400

設定の結果が一覧で表示されます。画面右上 ``Edit Configuration`` から設定の変更します。
Security Configuration 欄 右上の ``Show Advanced Fields`` をクリックします。
``API Definitions`` の ``Add Item`` をクリックします。新規作成のため、 ``Create new API Definition`` をクリックします

   .. image:: ./media/dcs-waap-lb-api-definition.jpg
       :width: 400

``Name`` 欄に API Definition の ``demo-app-api-definition`` を入力します。
Swagger Specs の欄に先程ImportしたSwagger FileのURLを入力します。 ``Add Item`` で入力欄を追加し、双方のURLを入力し、 ``Continue`` をクリックします

   .. image:: ./media/dcs-waap-lb-api-definition2.jpg
       :width: 400

一旦HTTP Load Blancerの設定を完了するため、最下部の ``Save & Exit`` をクリックし、設定を保存してください。

3. 作成した API Definition の確認
----

今回のサンプルでは2つのSwagger FileをImportしています。その2つのFileがどのような形でImportされ、またObjectが生成されているか確認します
``Web App & API Protection`` の画面左側 Manage欄、 ``API Management`` 、 ``API Definition`` を開き、作成したオブジェクト ``...`` から ``Show Child Objects`` をクリックしてください

   .. image:: ./media/dcs-waap-swagger-childobjects.jpg
       :width: 400

API Definitionで生成される、Child Objectsが表示されます。
今回の設定例では、2つのObjectsの名称が必要となりますので、 ``それぞれの名称をメモ`` してください。

   .. image:: ./media/dcs-waap-swagger-childobjects2.jpg
       :width: 400

ImportしたSwagger Fileと生成されたConfiguration Objectの詳細については Tips1 を参照してください


4. Service Policy の割当
----

``Service Policies`` を用いて、API の Access Control を設定します。
``ML Config`` ですが、本機能では使用しませんので、 ``Single ...`` から ``Multi ...`` と変更いただいても問題ありません。

再度HTTP LoadBalancerの設定を編集します。
画面上部、 ``Servgice Policies`` で ``Apply Specified Service Policies`` を選択し、 ``Configure`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy.jpg
       :width: 400

``List of Policy`` の ``Select Service policy`` から ``Create new service policy`` をクリックしてください

   .. image:: ./media/dcs-waap-lb-service-policy2.jpg
       :width: 400

``Name`` 欄に ``demo-app-service-policy`` と入力します。
``Rules`` の ``Select Policy Rules`` で ``Custom Rule List`` を選択し、 ``Configure`` をクリックします。
この項目で、通信制御のRuleを複数設定します

   .. image:: ./media/dcs-waap-lb-service-policy3.jpg
       :width: 400

Rule作成画面が表示されます。 ``Add Item`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy-rule.jpg
       :width: 400

1つ目のRuleを作成します。

``Name`` 欄に ``demo-app-sp-rule1`` と入力し、 ``Configure`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy-rule_1.jpg
       :width: 400

許可ルールを作成するため、 ``Action`` で ``Allow`` を選択します。最下部に移動し、API Group 欄の ``Configure`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy-rule_1-2.jpg
       :width: 400

先程コピーしたAPI Groupの名称のうち、 ``all-operations`` に該当するもの(この例では ``ves-io-api-def-demo-app-api-definition-all-operations`` )をコピーします。
Ruleの編集を完了するため、画面右下の ``Apply`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy-rule_1-3.jpg
       :width: 400

Rule の作成を完了するため、 ``API Group Matcher`` 、 ``Rule`` 双方の画面右下 ``Apply`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy-rule_1-4.jpg
       :width: 400

2つ目のRuleを作成します。

``Name`` 欄に ``demo-app-sp-rule2`` と入力し、 ``Configure`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy-rule_2.jpg
       :width: 400

拒否ルールを作成するため、 ``Action`` で ``Deny`` を選択します。最下部に移動し、API Group 欄の ``Configure`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy-rule_2-2.jpg
       :width: 400

先程コピーしたAPI Groupの名称のうち、 ``base-urls`` に該当するもの(この例では ``ves-io-api-def-demo-app-api-definition-base-urls`` )をコピーします。
Ruleの編集を完了するため、画面右下の ``Apply`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy-rule_2-3.jpg
       :width: 400

Rule の作成を完了するため、 ``API Group Matcher`` 、 ``Rule`` 双方の画面右下 ``Apply`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy-rule_2-4.jpg
       :width: 400

3つ目のRuleを作成します。

``Name`` 欄に ``demo-app-sp-rule3`` と入力し、 ``Configure`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy-rule_3.jpg
       :width: 400

すべてを許可ルールを作成するため、 ``Action`` で ``Allow`` を選択します。最下部に移動し、API Group 欄の ``Configure`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy-rule_3-2.jpg
       :width: 400

すべての通信を許可するルールのため、API Groupの名称は指定しません。
Rule の作成を完了するため、 ``API Group Matcher`` 、 ``Rule`` 双方の画面右下 ``Apply`` をクリックします

   .. image:: ./media/dcs-waap-lb-service-policy-rule_3-3.jpg
       :width: 400

以下のようにService Policyが作成されます。

   .. image:: ./media/dcs-waap-lb-service-policy-rule2.jpg
       :width: 400

表にまとめると以下の内容となります。

= ================= =======================================================
1 demo-app-sp-rule1 ``all-operations`` の API Group に該当する通信を ``許可``
2 demo-app-sp-rule2 ``base-urls`` の API Group に該当する通信を ``拒否``
3 demo-app-sp-rule3 すべての通信を ``許可``
= ================= =======================================================

画面右下のボタンを順次クリックし、設定を完了します

   .. image:: ./media/dcs-waap-lb-service-policy-rule3.jpg
       :width: 400


5. 動作確認
----

``all-operations`` の API Group に該当するリクエストをCurlコマンドで実施し、通信が ``許可`` されることが確認できます

.. code-block:: bash
  :linenos:
  :caption: Curl コマンドを使った https://echoapp.f5demo.net/rest/basket/1 への接続結果  

  $ curl -ks https://echoapp.f5demo.net/rest/basket/1
  {"request":{"headers":[["host","app1.test10demo.xyz"],["user-agent","curl/7.58.0"],["accept","*/*"],["x-forwarded-for","18.178.83.1"],["x-forwarded-proto","https"],["x-envoy-external-address","18.178.83.1"],["x-request-id","33a40044-32b4-4e8e-8705-ea0e351d0c75"],["content-length","0"]],"status":0,"httpversion":"1.1","method":"GET","scheme":"http","uri":"/rest/basket/1","requestText":"","fullPath":"/rest/basket/1"},"network":{"clientPort":"49244","clientAddress":"103.135.56.118","serverAddress":"192.168.16.2","serverPort":"80"},"ssl":{"isHttps":false},"session":{"requestId":"872c2a9a09cad3dd53d61df4ce216178","connection":"7","connectionNumber":"1"},"environment":{"hostname":"echoapp"}}


``base-urls`` の API Group に該当するリクエストをCurlコマンドで実施し、通信が ``拒否`` されることが確認できます

.. code-block:: bash
  :linenos:
  :caption: Curl コマンドを使った https://echoapp.f5demo.net/rest/ への接続結果  

  $  curl -vks https://echoapp.f5demo.net/rest/
  
  ** 省略 **
  
  <h1>
  Error 403 - Forbidden
  </h1>

ブラウザでアクセスした場合には以下のようにエラーが確認できます

   .. image:: ./media/dcs-waap-api-service-policy-browser.jpg
       :width: 400

以下リクエストは3つ目のルールに該当します。Curlコマンドでリクエストを送付し、通信が ``許可`` されることが確認できます

.. code-block:: bash
  :linenos:
  :caption: Curl コマンドを使った https://echoapp.f5demo.net/others への接続結果  

  $ curl -ks https://echoapp.f5demo.net/others
  {"request":{"headers":[["host","app1.test10demo.xyz"],["user-agent","curl/7.58.0"],["accept","*/*"],["x-forwarded-for","18.178.83.1"],["x-forwarded-proto","https"],["x-envoy-external-address","18.178.83.1"],["x-request-id","31e50ded-03cd-4bb5-b514-03fea51cc18b"],["content-length","0"]],"status":0,"httpversion":"1.1","method":"GET","scheme":"http","uri":"/others","requestText":"","fullPath":"/others"},"network":{"clientPort":"33739","clientAddress":"103.135.56.106","serverAddress":"192.168.16.2","serverPort":"80"},"ssl":{"isHttps":false},"session":{"requestId":"d05e00c647ead07c37f2bb0d6aad3f69","connection":"6","connectionNumber":"1"},"environment":{"hostname":"echoapp"}}




Tips1. Swagger File と Configuration Objectの詳細
----

次に、 :download:`REST API Swagger File <./file/rest-api.json>` の内容と生成された Child Object の内容を確認します。

.. code-block:: json
  :linenos:
  :caption: REST API Swagger File
  :emphasize-lines: 8,14,40,54

  {
      "swagger": "2.0",
      "info": {
        "description": "Juice Shop REST",
        "title": "Juice Shop REST",
        "version": "v1"
      },
      "basePath": "/rest",
      "schemes": [
        "http",
        "https"
      ],
      "paths": {
        "/basket/{id}": {
          "get": {
            "consumes": [
              "application/json"
            ],
            "description": "Swagger auto-generated from learnt schema",
            "parameters": [
              {
                "name": "id",
                "in": "path",
                "description": "ID",
                "required": true,
                "type": "integer",
                "format": "int64"
              }
            ],
            "responses": {
              "200": {
                "description": ""
              }
            }
          }
        },
                
        ** 省略 **
        
        "/wallet/balance": {
         "get": {
            "consumes": [
              "application/json"
            ],
            "description": "Swagger auto-generated from learnt schema",
            "parameters": [
              
            ],
            "responses": {
              "200": {
                "description": ""
              }
            },
            "x-volterra-api-group":"sensitive"
          }
        },
                
        ** 省略 **

- 8行目 basePath ``/rest`` であることが確認できます
- 14行目 path ``/basket/{id}`` であることが確認できます
- 54行目 ``x-volterra-api-group`` でAPI Groupを指定することが可能です。この例では、 ``sensitive`` というAPI Groupを指定しています
- 40行目 path ``/wallet/balance`` は54行目の内容により、 ``sensitive`` のAPI Groupとするよう指定しています

``base-urls`` の API Group を確認します。

.. code-block:: json
  :linenos:
  :caption: API Group (ves-io-api-def-demo-app-api-definition-base-urls)
  :emphasize-lines: 3,28      

  {
    "metadata": {
      "name": "ves-io-api-def-demo-app-api-definition-base-urls",
      "namespace": "h-matsumoto",
      "labels": {
        "ves.io/api-scope": "ves-io-demo-app-api-definition"
      },
        
    ** 省略 **
    
    "spec": {
      "elements": [
        
      ** 省略 **
    
        {
          "methods": [
            "GET",
            "HEAD",
            "POST",
            "PUT",
            "DELETE",
            "CONNECT",
            "OPTIONS",
            "TRACE",
            "PATCH"
          ],
          "path_regex": "^/rest/.*$"
        }
      ]
    },
     
  ** 省略 **

- 28行目の内容を確認すると、 ``REST API Swagger File`` の 8行目 basePath の内容が確認できます

``all-operations`` の API Group を確認します。

.. code-block:: json
  :linenos:
  :caption: API Group (ves-io-api-def-demo-app-api-definition-all-operations)
  :emphasize-lines: 3,20  

  {
    "metadata": {
      "name": "ves-io-api-def-demo-app-api-definition-all-operations",
      "namespace": "h-matsumoto",
      "labels": {
        "ves.io/api-scope": "ves-io-demo-app-api-definition"
      },
     
    ** 省略 **
  
    "spec": {
      "elements": [
     
      ** 省略 **
  
        {
          "methods": [
            "GET"
          ],
          "path_regex": "^/rest/basket/([\\w\\-._~%!$&'()*+,;=:]+)$"
        }
      ]
    },
     
  ** 省略 **

- 28行目の内容を確認すると、basePath ``/rest`` に ``REST API Swagger File`` の 14行目 path を追加した内容が確認できます

.. code-block:: json
  :linenos:
  :caption: API Group (ves-io-api-def-demo-app-api-definition-sensitive)
  :emphasize-lines: 3,17      

  {
    "metadata": {
      "name": "ves-io-api-def-demo-app-api-definition-sensitive",
      "namespace": "h-matsumoto",
      "labels": {
        "ves.io/api-scope": "ves-io-demo-app-api-definition"
      },
                  
      ** 省略 **
  
    "spec": {
      "elements": [
        {
          "methods": [
            "GET"
          ],
          "path_regex": "^/rest/wallet/balance$"
        },
        {
          "methods": [
            "GET"
          ],
          "path_regex": "^/rest/user/whoami$"
        }
      ]
    },
                  
  ** 省略 **

- 3行目の通り、 ``REST API Swagger File`` の 54行目 ``sensitive`` の名称で API Group が作成されています
- 28行目の内容を確認すると、basePath ``/rest`` に ``REST API Swagger File`` の 40行目 path を追加した内容が確認できます

4. XC API Security の解除
====

その他の機能を確認するため設定を解除する手順です。HTTP Load Balancerに割り当てたAPI Security に関連する設定を解除してください

   .. image:: ./media/dcs-single-api-security-disable.jpg
       :width: 400


5. Terraform を用いた HTTP Load Balancer + API Security の利用
====

HTTP Load Balancer + API Discovery の作成
----

ここで紹介したHTTP load Balancer + API Discovery を Terraform を使ってデプロイすることが可能です。

Terraform の利用で必要となる事前作業については `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module03/module03.html>`__ の手順を参考してください

パラメータの指定
~~~~

実行に必要なファイル、また実行環境に合わせたパラメータを指定してください

.. code-block:: bash
  :linenos:
  :caption: terraform 実行前作業

  $ git clone https://github.com/BeF5/f5j-dc-waap-automation
  $ cd f5j-dc-waap-automation/terraform/api-discovery

  $ vi terraform.tfvars
  # ** 環境に合わせて適切な内容に変更してください **
  api_p12_file     = "**/path/to/p12file**"        // Path for p12 file downloaded from VoltConsole
  api_url          = "https://**api url**"     // API URL for your tenant

  # 本手順のサンプルで表示したパラメータの場合、以下のようになります 
  myns             = "**your namespace**"      // Name of your namespace
  op_name          = "demo-origin-pool"        // Name of Origin Pool
  pool_port        = "80"                      // Port Number
  server_name1     = "**your target fqdn1**"   // Target Server FQDN1
  server_name2     = "**your target fqdn1**"   // Target Server FQDN2
  httplb_name      = "demo-echo-lb"            // Name of HTTP LoadBalancer
  mydomain         = ["echoapp.f5demo.net"]    // Domain name to be exposed
  
  cert             = "string///**base 64 encode SSL Certificate**"  // SSL Certificate for HTTPS access
  private_key      = "string///**base 64 encode SSL Private Key**"  // SSL Private Key for HTTPS access

Terraform の利用
~~~~

以下コマンドを参考に実行および削除をしてください。

.. code-block:: bash
  :linenos:
  :caption: terraform の実行・削除

  # 実行前事前作業
  $ terraform init
  $ terraform plan

  # 設定のデプロイ
  $ terraform apply

  # 設定の削除
  $ terraform destroy

HTTP Load Balancer + API Definitionを用いた通信制御
----

Swagger FileのImport及び、API DefinitionはコンソールよりGUIで設定する必要があります。
`こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module10/module10.html#swagger-file-api-group>`__ の手順に従って操作をしてください。
``3. API Definition の作成`` のマニュアルはHTTP load Balancerから設定する手順としていますが、個別に作成する場合には以下手順に従って、 ``API Definition`` を作成してください


メニューより ``Web App & API Protection`` を開いてください。

   .. image:: ./media/dcs-console-waap.jpg
       :width: 400

画面左側 Manage欄の ``API Management`` 、 ``API Definition`` を開き、 ``Add API Definition`` より新規作成してください

   .. image:: ./media/dcs-waap-api-definition.jpg
       :width: 400

``Name`` 欄に API Definition の ``demo-app-api-definition`` を入力します。
Swagger Specs の欄に先程ImportしたSwagger FileのURLを入力します。 ``Add Item`` で入力欄を追加し、双方のURLを入力し、 ``Continue`` をクリックします

   .. image:: ./media/dcs-waap-lb-api-definition2.jpg
       :width: 400

.. NOTE::
  Terraform のサンプルファイルは、API Definition の名称が ``demo-app-api-definition`` という想定となっております。
  API Definition の名称が異なる場合、生成されるChild Objectの名称も異なるため、 ``all-operations`` 、 ``base-urls`` 等に関連する名称を適切に変更してください。

パラメータの指定
~~~~

実行に必要なファイル、また実行環境に合わせたパラメータを指定してください

.. code-block:: bash
  :linenos:
  :caption: terraform 実行前作業

  $ git clone https://github.com/BeF5/f5j-dc-waap-automation
  $ cd f5j-dc-waap-automation/terraform/api-control

  $ vi terraform.tfvars
  # ** 環境に合わせて適切な内容に変更してください **
  api_p12_file     = "**/path/to/p12file**"        // Path for p12 file downloaded from VoltConsole
  api_url          = "https://**api url**"     // API URL for your tenant

  # 本手順のサンプルで表示したパラメータの場合、以下のようになります 
  myns             = "**your namespace**"      // Name of your namespace
  op_name          = "demo-origin-pool"        // Name of Origin Pool
  pool_port        = "80"                      // Port Number
  server_name1     = "**your target fqdn1**"   // Target Server FQDN1
  server_name2     = "**your target fqdn1**"   // Target Server FQDN2
  httplb_name      = "demo-echo-lb"            // Name of HTTP LoadBalancer
  mydomain         = ["echoapp.f5demo.net"]    // Domain name to be exposed
  
  cert             = "string///**base 64 encode SSL Certificate**"  // SSL Certificate for HTTPS access
  private_key      = "string///**base 64 encode SSL Private Key**"  // SSL Private Key for HTTPS access

  // Service Policy Parameter
  sp_name          = "demo-app-service-policy"

Terraform の利用
~~~~

以下コマンドを参考に実行および削除をしてください。

.. code-block:: bash
  :linenos:
  :caption: terraform の実行・削除

  # 実行前事前作業
  $ terraform init
  $ terraform plan

  # 設定のデプロイ
  $ terraform apply

  # 設定の削除
  $ terraform destroy

6. API を用いた HTTP Load Balancer + API Security の利用
====

ここで紹介したHTTP load Balancer + API Security を XC の API を使ってデプロイすることが可能です。

API の利用で必要となる事前作業については `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module03/module03.html>`__ の手順を参考してください

以下マニュアルを参考に、パラメータを指定して実行してください。

- Service Policy

  - `API for service_policy <https://docs.cloud.f5.com/docs/api/service-policy>`__
  - `Example of creating service_policy <https://docs.cloud.f5.com/docs/reference/api-ref/ves-io-schema-service_policy-api-create>`__

    - ページ中段 ``Request using curl`` をご覧ください

作成したオブジェクトを適宜HTTP Load Balancerから参照してください

- HTTP Load Balancer

  - `API for http_loadbalancer <https://docs.cloud.f5.com/docs/api/views-http-loadbalancer>`__
  - `Example of creating http_loadbalancer <https://docs.cloud.f5.com/docs/reference/api-ref/ves-io-schema-views-http_loadbalancer-api-create>`__

    - ページ中段 ``Request using curl`` をご覧ください

送付するJSON データの書式は実際に作成したコンフィグのJSONデータからも確認をいただけます。合わせてご確認ください

HTTP Load Balancer + API Discovery の作成
----

パラメータの指定
~~~~

GitHubよりファイルを取得します。 ``api-discovery-httplb.json`` をAPIの値として指定します。
``**<変数名>**`` が環境に合わせて変更するパラメータとなります。適切な内容に変更してください。

``Originl Pool Object`` は HTTP Load Balancer の Originl Pool 作成手順に従って作成ください

APIの利用
~~~~

以下のサンプルを参考にAPIを実行してください。
証明書のファイル名、パスワード情報は適切な内容を指定してください。

- ファイル取得

.. code-block:: bash
  :linenos:
  :caption: APIによるオブジェクトの作成

  $ git clone https://github.com/BeF5/f5j-dc-waap-automation
  $ cd f5j-dc-waap-automation/api/api-discovery

- オブジェクトの作成

.. code-block:: bash
  :linenos:
  :caption: APIによるオブジェクトの作成 (HTTP Load Balancer の Origin Pool 設定ファイルを指定)

  # Originl Pool の作成 (HTTP LoadBalancer のパラメータを指定)
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/origin_pools \
       --cert **/path/to/api_credential.p12-file**:**password** \
       --cert-type P12 \
       -X POST \
       -d @../http-load-balancer/base-origin-pool.json

  # HTTP LB の作成
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/http_loadbalancers \
       --cert **/path/to/api_credential.p12-file**:**password** \
       --cert-type P12 \
       -X POST \
       -d @api-discovery-httplb.json


- オブジェクトの削除

.. code-block:: bash
  :linenos:
  :caption: APIによるオブジェクトの削除

  # HTTP LB の削除
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/http_loadbalancers/**httplb_name** \
       --cert **/path/to/api_credential.p12-file** \
       --cert-type P12 \
       -X DELETE
  
  # Origin Pool の削除
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/origin_pools/**op_name** \
       --cert **/path/to/api_credential.p12-file** \
       --cert-type P12 \
       -X DELETE

HTTP Load Balancer + API Definitionを用いた通信制御 の作成
----

パラメータの指定
~~~~

GitHubよりファイルを取得します。 ``api-control-httplb.json`` をAPIの値として指定します。
``**<変数名>**`` が環境に合わせて変更するパラメータとなります。適切な内容に変更してください。

| ``Originl Pool Object`` は HTTP Load Balancer の Originl Pool 作成手順に従って作成ください。
| ``App Firewall Object`` は WAF の App Firewall 作成手順に従って作成してください

Swagger File の Import及びAPI Definitionは別途GUIから作成が必要です。詳細は `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module10/module10.html#terraform-http-load-balancer-api-definition>`__ を参照してください。

APIの利用
~~~~

以下のサンプルを参考にAPIを実行してください。
証明書のファイル名、パスワード情報は適切な内容を指定してください。

- ファイル取得

.. code-block:: bash
  :linenos:
  :caption: APIによるオブジェクトの作成

  $ git clone https://github.com/BeF5/f5j-dc-waap-automation
  $ cd f5j-dc-waap-automation/api/api-control
  
- オブジェクトの作成

.. code-block:: bash
  :linenos:
  :caption: APIによるオブジェクトの作成

  # Originl Pool の作成 (HTTP LoadBalancer のパラメータを指定)
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/origin_pools \
       --cert **/path/to/api_credential.p12-file**:**password** \
       --cert-type P12 \
       -X POST \
       -d @../http-load-balancer/base-origin-pool.json

  # Service Policy の作成
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/service_policys \
       --cert **/path/to/api_credential.p12-file** \
       --cert-type P12 \
       -X POST \
       -d @api-control-service-policy.json

  # HTTP LB の作成
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/http_loadbalancers \
       --cert **/path/to/api_credential.p12-file** \
       --cert-type P12 \
       -X POST \
       -d @api-control-httplb.json


- オブジェクトの削除

.. code-block:: bash
  :linenos:
  :caption: APIによるオブジェクトの削除

  # HTTP LB の削除
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/http_loadbalancers/**httplb_name** \
       --cert **/path/to/api_credential.p12-file** \
       --cert-type P12 \
       -X DELETE

  # Service Policy の削除
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/service_policys/**sp_name**  \
       --cert **/path/to/api_credential.p12-file** \
       --cert-type P12 \
       -X DELETE

  # Origin Pool の削除
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/origin_pools/**op_name** \
       --cert **/path/to/api_credential.p12-file** \
       --cert-type P12 \
       -X DELETE
