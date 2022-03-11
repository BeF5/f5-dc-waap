F5 DCS HTTP LB 各種追加設定
####

Load Balancer で利用する各種設定について紹介します

マニュアルは以下のページを参照してください
- `HTTP Load Balancer <https://docs.cloud.f5.com/docs/how-to/app-networking/http-load-balancer>`__

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

2. HTTP Load Balancer の基本的な設定項目
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

   .. image:: ./media/dcs-setting-lb-route.jpg
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


3. HTTP LB 追加設定
====

1. Health Checkの追加
----

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

2. Origin Poolの追加
----

画面左側、 ``Load Balancers`` 、 ``Origin Pools`` から一覧を表示し、 ``Add Origin Pool`` をクリックします

   .. image:: ./media/dcs-setting-origin.jpg
       :width: 400

基本的な設定内容はすでに設定の通りです。Origin Pool はRouteなど、特定のURL Pathに通信が発生した場合の転送先として指定することが可能です。
各Origin Poolでは通信の転送に関わる各種設定を行うことが可能です。

   .. image:: ./media/dcs-setting-origin2.jpg
       :width: 400

Tips1. HTTP Load Balancer 作成時にシステムが生成される Child Object 
====

HTTP Load Balancer を設定すると、同Namespace内に生成されるObjectの他に自動的に Child Object が生成される場合があります

1. Child Object の確認
----

シンプルなHTTP Load Balancerの設定のChild Objectを確認します

すでに作成済みのオブジェクトを変更する場合、対象のオブジェクト一番右側 ``‥`` から、 ``Show Child Objects`` をクリックします

   .. image:: ./media/dcs-setting-childobjects.jpg
       :width: 400

以下のような画面が表示されます。この設定では、 ``Route`` 、 ``Virtual host`` 、 ``Advertise policy`` が確認できます。
設定内容はJSON形式で表示されています。

   .. image:: ./media/dcs-setting-childobjects2.jpg
       :width: 400

JSON の内容を確認すると、namespace は HTTP Load Balancer と同一となっていますが、設定画面上はこれらの内容は Object として個別に表示はされません。あくまでChild ObjectとしてこのHTTP Load Balancerのみで利用されます
対し、以下のようなML Configを設定します。


2. ML Config に関連するオブジェクトと生成される内容
----

Malicious User などで利用するML Configを利用します。
以下が設定例となります。

   .. image:: ./media/dcs-mlconfig-sample.jpg
       :width: 400

設定を反映するため ``Save and Exit`` をクリックした後、再度設定を開くと ``Label`` が自動的に付与されていることが確認できます。

   .. image:: ./media/dcs-mlconfig-sample2.jpg
       :width: 400

Single Load Balancerで設定した場合、上記の例と同様にChild Objectが生成されます。
先程のHTTP Load Balancerの設定に加え、 ``App type`` と ``App Setting`` が生成されていることが確認できます

   .. image:: ./media/dcs-setting-childobjects3.jpg
       :width: 400


Multi の HTTP Load Balancerに対する設定を行う場合、別途 ``Shared Configuration`` から ``AI & ML`` の ``app_type`` で
パラメータを指定し、HTTP Load Balancerの ``Label`` で紐付けを指定します。
Single ではこれらの内容が自動的に生成、反映される動作となっていることが確認できます。

