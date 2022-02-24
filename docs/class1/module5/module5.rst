F5 DCS WAF
####

WAF で利用する各種設定について紹介します

1. 基本的な既存設定の変更方法
====

すでに作成済みのオブジェクトを変更する場合、対象のオブジェクト一番右側 ``‥`` から、 ``Manage Configuration`` をクリックします

   .. image:: ./media/dcs-setting-edit.jpg
       :width: 400

設定の結果が一覧で表示されます。画面右上 ``Edit Configuration`` から設定の変更が可能です

   .. image:: ./media/dcs-setting-edit2.jpg
       :width: 400

変更画面は設定の新規作成画面と同様です。設定の変更を行わない場合、左下の ``Cancel and Exit`` をクリックし、中断できます

   .. image:: ./media/dcs-setting-edit3.jpg
       :width: 400

2. Health Checkの追加
====

Health Check ルールを追加することにより、Origin Poolに指定したServerの障害を回避します

画面左側、 ``Load Balancers`` 、 ``Health Checks`` から一覧を表示し、 ``Add health check`` をクリックします

   .. image:: ./media/dcs-setting-hc.jpg
       :width: 400

追加するHealth Checkの名称を指定し、画面中段から意図した設定となるようにパラメータを指定します。
``HTTP HealthCheck`` を選択した例となりますが、 ``Configure`` をクリックし、詳細のパラメータを指定します

   .. image:: ./media/dcs-setting-hc2.jpg
       :width: 400

以下が ``Configure`` から遷移する詳細画面です。内容を指定し、 ``Apply`` をクリックします

   .. image:: ./media/dcs-setting-hc3.jpg
       :width: 400

その他の、内容を指定し、 ``save and Exit`` をクリックします

   .. image:: ./media/dcs-setting-hc4.jpg
       :width: 400

3. Origin Poolの追加
====

画面左側、 ``Load Balancers`` 、 ``Origin Pools`` から一覧を表示し、 ``Add Origin Pool`` をクリックします

   .. image:: ./media/dcs-setting-origin.jpg
       :width: 400

基本的な設定内容はすでに設定の通りです。Origin Pool はRouteなど、特定のURL Pathに通信が発生した場合の転送先として指定することが可能です。
各Origin Poolでは通信の転送に関わる各種設定を行うことが可能です。

   .. image:: ./media/dcs-setting-origin2.jpg
       :width: 400

4. HTTP Load Balancer の設定項目
====

HTTP Load Balancer は各種通信のリクエスト、レスポンスに関する制御を指定します。
HTTP Load Balancer で利用する各種設定項目について紹介します

1. Basic Configuration
----

通信を待ち受けるために必要となる設定を行います

   .. image:: ./media/dcs-setting-lb-basic.jpg
       :width: 400

   .. image:: ./media/dcs-setting-lb-basic.jpg
       :width: 400


2. Route
----

Pathに応じたより詳細な転送方法をしていします。このRouteではこの項目で紹介する多くのその他詳細設定も含め、Path毎の細かな通信制御を行うことが可能です

   .. image:: ./media/dcs-setting-lb-route1.jpg
       :width: 400

   .. image:: ./media/dcs-setting-lb-route2.jpg
       :width: 400

3. VIP Configuration
----

通信を受け付けるIPアドレスの指定方法などの設定を行います

   .. image:: ./media/dcs-setting-lb-vip.jpg
       :width: 400

4. Security Configuration
----

各種セキュリティに関する設定を行います

   .. image:: ./media/dcs-setting-lb-security.jpg
       :width: 400


4. Load Balancing Control
----

Load Balance Algorithm の指定や、その他制御方法に関する設定を行います

   .. image:: ./media/dcs-setting-lb-lbcontrol.jpg
       :width: 400

5. Advanced Configuration
----

その他各種詳細の設定を行います

   .. image:: ./media/dcs-setting-lb-advanced.jpg
       :width: 400


