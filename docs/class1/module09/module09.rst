F5 DCS Malicious User Detection & Mitigation
####

F5 DCS Malicious User Detection & Mitigation を利用する方法や、各種設定について紹介します

マニュアルは以下のページを参照してください

- `Malicious Users <https://docs.cloud.f5.com/docs/how-to/advanced-security/malicious-users>`__


1. F5 DCS Malicious User Detection & Mitigation について
====

F5 DCS Malicious User Detection & Mitigationは、以下の特徴を備えております

- サイトへのアクセスからユーザを識別し、ある特定のユーザのセキュリティ侵害や攻撃トラフィック、またJavaScript や Captcha の結果など、様々な観点から調査し、そのユーザのふるまいから悪意あるユーザであるか判定
- 識別したユーザの通信をダッシュボードで確認
- 対象のユーザからの通信をブロック
- Malicious User の機能は、F5 DCSが提供する AI & ML の機能を利用しています。Malicious User Detection の設定を単一(Single)の Load Balancer で利用する場合と、複数(Multi)の Load Balancer で利用する場合で設定方法が異なる

F5 DCS WAAPはこれらの高度なセキュリティをアプリケーションの迅速な展開に合わせて自由にご利用いただける環境を実現します

2. Single LB で Malicious User の設定
====

単一のLoad Balancerで設定する方法を示します

1. Malicious User Detection & Mitigation の設定
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

動作確認するクライアントの通信を Malicious User として判定するため、App Firewallを用いて通信をブロックします。
以前作成した App Firewall のポリシーを割り当てます。（App Firewallの操作手順は `F5 DCS WAF <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module05/module05.html>`__ を参照してください。

   .. image:: ./media/dcs-edit-lb-malicious-user-waf.jpg
       :width: 400

Malicious User Mitigation の方法を指定します。
``Select Type of Challenge`` で ``Policy Based Challenge`` を選択し、 ``Configure`` をクリックしてください。
``Policy Based Challenge`` の設定が表示されますので、内容を変更せず初期設定のまま ``Apply`` をクリックしてください。

   .. image:: ./media/dcs-edit-lb-malicious-user.jpg
       :width: 400

Malicious User Detection の方法を指定します。
今回は、単一のLoad Balancerを対象とした設定となりますので、 ``ML Config`` で ``Single Load Balancer Application`` を選択します。
その配下に表示される ``Malicious User Detection`` で ``Enable Malicious User Detection`` を選択してください。
その他機能は利用しませんので、 ``無効 (Disable)`` を選択してください

   .. image:: ./media/dcs-edit-lb-single-malicious-user.jpg
       :width: 400

正しく設定されたことを確認し、画面最下部の ``Apply`` をクリックしてください。

   .. image:: ./media/dcs-edit-lb-malicious-user-apply.jpg
       :width: 400


3. Single LB での Malicious User の動作確認
====

1. Curlコマンドによる Malicious User の確認
----

以下Curlコマンドを実行します。1秒毎にクロスサイトスクリプティング(XSS)として検知されるリクエストを送付します。

.. code-block:: bash
  :linenos:
  :caption: Curl コマンドを使った https://echoapp.f5demo.net への接続結果

  $ while : ; do sleep 1 ; date ; curl -ks "https://echoapp.f5demo.net/?<script>"  ; done


一定時間、コマンドを実行してください。
次の項目からステータスの確認について説明します。これらの内容が確認できれば、 ``Ctrl-C`` でコマンドを停止させてください

2. Security Event の確認
----

以下の手順で Security Event を開いてください

   .. image:: ../module06/media/dcs-app-fw-log.jpg
       :width: 400

   .. image:: ../module06/media/dcs-app-fw-log2.jpg
       :width: 400

最新の情報を取得するため、画面右上 ``Refresh`` をクリックしてください。
画面中段で ``Security Events`` が選択され、下に ``WAF events`` の一覧が表示されていることを確認してください。
いくつかログが表示されており、XSS を検知していることがわかります。

   .. image:: ./media/dcs-malicious-user-log.jpg
       :width: 400

詳細は別途 `F5 DCS WAF <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module06/module06.html>`__ を参照ください。

画面中段の ``Malicious User Events`` が選択してください。
下に ``Malicious User events`` の一覧が表示されていることを確認してください。

   .. image:: ./media/dcs-malicious-user-log2.jpg
       :width: 400

どのようなユーザが、どのようなイベントで検知されたか確認することができます。

JSONの表示内容は以下のとおりです。

.. code-block:: json
  :linenos:
  :caption: Malicious User waf_sec_event
  :emphasize-lines: 4,8,9,38,43

  {
    "country": "JP",
    "kubernetes": {},
    "app_type": "ves-io-http-loadbalancer-demo-echo-lb",
    "summary_msg": "4 WAF security events in last 15 seconds.",
    "waf_sec_event_count": 100,
    "failed_login_suspicion_score": 0,
    "mitigation_activity_info": "{\"mum_captcha_challenge\":10,\"mum_js_challenge\":0,\"mum_temporarily_blocking\":0}",
    "incremental_activity_info": "{\"err_count\":0,\"failed_login_count\":0,\"forbidden_access_count\":0,\"req_count\":14,\"waf_sec_event_count\":4}",
    "suspicion_log_type": "detection",
    "hostname": "master-1",
    "req_count": 2286,
    "tenant": "f5-apac-ent-uppdoshj",
    "longitude": "139.689900",
    "app": "obelix",
    "source_type": "kafka",
    "pdf_inference_score": [
      0,
      0,
      0,
      0
    ],
    "start_time": 1646835501,
    "feature_score": "{}",
    "gmm_anomaly": 0,
    "err_count": 798,
    "region": "13",
    "city": "Tokyo",
    "latitude": "35.689300",
    "messageid": "6f2a6baa-3a2c-470f-a0ec-527c63f5c723",
    "method_counts": "{\"CONNECT\":0,\"DELETE\":0,\"GET\":2151,\"HEAD\":0,\"OPTIONS\":0,\"PATH\":0,\"POST\":135,\"PUT\":0,\"TRACE\":0}",
    "smg_anomaly": 0,
    "network": "18.176.0.0",
    "src_ip": "18.178.83.1",
    "failed_login_count": 0,
    "forbidden_access_suspicion_score": 0,
    "stream": "svcfw",
    "suspicion_score": 1,
    "message_key": null,
    "severity": "info",
    "cluster_name": "ty8-tky-int-ves-io",
    "headers": {},
    "threat_level": "High",
    "types": "input:string",
    "ip_reputation_suspicion_score": 0,
    "behavior_anomaly_score": 0,
    "end_time": 1646835516,
    "apiep_anomaly": 0,
    "message": "User Suspicion Score",
    "site": "ty8-tky",
    "@timestamp": "2022-03-09T14:18:36.042Z",
    "forbidden_access_count": 31,
    "namespace": "h-matsumoto",
    "time": "2022-03-09T14:18:36.042Z",
    "asn": "AMAZON-02(16509)",
    "sec_event_type": "malicious_user_sec_event",
    "user": "IP-18.178.83.1",
    "vh_name": "ves-io-http-loadbalancer-demo-echo-lb",
    "waf_suspicion_score": 1
  }

- 4行目にメッセージの概要が表示されています
- 8行目、9行目では、このイベントでどのような ``Activity`` が行われているのか確認できます
- 38行目では ``suspicion_score`` 、43行目では ``threat_level`` が表示されています

その他にも多くの情報が記載されておりますので、適宜参照してください。


3. Malicious Users の確認
----

Security Event は主に時系列でのイベントを表示しています。
Malicious Users では、Malicious User と判定されたユーザ毎にどのような判定がなされたのかその経緯を俯瞰的に確認することが可能です。

画面上部 ``Malicious Users`` をクリックしてください。
この例では、画面左側 ``Malicious User`` が1つで、そのアクティビティの詳細が右側に表示されます。
複数の Malicious User が検知されている場合には複数表示されます

   .. image:: ./media/dcs-malicious-user-log3.jpg
       :width: 400

各エントリにリンクとして表示される項目をクリックすると、項目に該当する期間のEventを確認することができます

また実際にブロックされた場合には、``Error 403 Forbidden`` という形で通信がブロックされます

.. code-block:: bash
  :linenos:
  :caption: (参考) ブロックされた場合の応答結果

  > GET /?<script> HTTP/2
  > Host: echoapp.f5demo.net
  > User-Agent: curl/7.58.0

  ** 省略 **

  < HTTP/2 403
  < content-length: 27653
  < content-type: text/html; charset=UTF-8
  
  ** 省略 **

  <h1>
  Error 403 - Forbidden
  </h1>

4. ブラウザによる Malicious User の確認
----

ブラウザで複数回、継続して攻撃に相当するリクエストを送ることで Malicious User として検知することが可能です。

ブラウザで、攻撃として検知されるURL ``https://echoapp.f5demo.net/?<script>`` にアクセスし、連続して一定時間ページの更新を行ってください。

攻撃がブロックされると以下のような画面が表示されます

   .. image:: ./media/dcs-malicious-user-browser.jpg
       :width: 400


4. JS / Captcha Challenge の確認
====

JavaScript や Captcha を用いて Malicious User であるかどうか検査することが可能です。
以下に動作確認を目的としたパラメータの変更方法と動作確認の結果を紹介します。

1. Challenge Type の選択
----

HTTP Load Balancer の設定変更します。
Security Configuration 欄 右上の ``Show Advanced Fields`` をクリックします。

先程設定した Challenge に関する設定を変更します。
``Select Type of Challenge`` の ``Policy Based Challenge`` 下の ``Edit Configuration`` をクリックしてください。

   .. image:: ./media/dcs-edit-lb-malicious-user-challenge.jpg
       :width: 400

表示された画面の ``Select Type of Challenge`` のプルダウンから項目が選択できます。


2. JS Challenge の設定・確認
----

Javascript による Challenge の動作を確認します。
プルダウンから ``Always enable JS Challenge`` を選択し、
画面最下部の ``Apply`` をクリックし、さらに、HTTP Load Balancer の ``Apply`` をクリックし設定を反映してください。

   .. image:: ./media/dcs-edit-lb-malicious-user-challenge2.jpg
       :width: 400

設定の反映後、ブラウザで ``https://echoapp.f5demo.net/`` にアクセスしてください。

``JS Challenge`` は、HTTP Load Balancer がリクエストに対し、Malicious User であるか判定するのに用いる JavaScript を応答し動作を確認します。
初期設定では以下のような画面が表示され、画面の案内の通り1秒後画面が本来期待したコンテンツへと切り代わります。

- JS Challenge の表示

   .. image:: ./media/dcs-edit-lb-malicious-js-challenge-browser.jpg
       :width: 400

- 正しくCpatchaが処理され、コンテンツが表示される

   .. image:: ./media/dcs-edit-lb-malicious-challenge-result.jpg
       :width: 400


3. Captcha Challenge の設定・確認
----

Captcha による Challenge の動作を確認します。
プルダウンから ``Always enable JS Challenge`` を選択し、
画面最下部の ``Apply`` をクリックし、さらに、HTTP Load Balancer の ``Apply`` をクリックし設定を反映してください。

   .. image:: ./media/dcs-edit-lb-malicious-user-challenge3.jpg
       :width: 400

設定の反映後、ブラウザで ``https://echoapp.f5demo.net/`` にアクセスしてください。

``Captcha Challenge`` は、HTTP Load Balancer がリクエストに対し、Malicious User であるか判定するのに用いる Captcha を応答し動作を確認します。
以下のような画面が表示されます。ユーザは自身がロボットではないことを証明するため、チェックボックスにチェックを入れた後、支持に従って画像を選択します。
その後、本来期待したコンテンツへと切り代わります。

- Captchaの表示

   .. image:: ./media/dcs-edit-lb-malicious-captcha-challenge-browser.jpg
       :width: 400

   .. image:: ./media/dcs-edit-lb-malicious-captcha-challenge-browser2.jpg
       :width: 400

- 正しくCpatchaが処理され、コンテンツが表示される

   .. image:: ./media/dcs-edit-lb-malicious-challenge-result.jpg
       :width: 400

.. NOTE::
  JS / Captcha Challengeがクライアントに表示される動作は、各種ログ ``Security Events`` や ``Malicious Users`` に都度記録されるものではありません


5. F5 DCS Malicious User Detection の解除
====

その他の機能を確認するための手順です。

`こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module09/module09.html#single-lb-malicious-user>`__ の手順を参考に、HTTP Load Balancerに割り当てたMalicious Userの設定を解除してください

   .. image:: ./media/dcs-single-malicious-user-disable.jpg
       :width: 400

6. Terraform を用いた HTTP Load Balancer + Malicious User Detection の作成
====

ここで紹介したHTTP load Balancer + Malicious User Detection を Terraform を使ってデプロイすることが可能です。

Terraform を用いた設定の作成方法については `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module03/module03.html>`__ の手順を参考してください

実行に必要なファイル、また実行環境に合わせたパラメータを指定してください

.. code-block:: bash
  :linenos:
  :caption: terraform 実行前作業

  $ git clone https://github.com/hiropo20/terraform-f5dcs-waap.git
  $ cd malicious-user-detection

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

  // WAF Parameter
  waf_name         = "demo-app-fw"            // Name of App Firewall

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