F5 DCS WAF
####

F5 DCS で WAF を利用する方法や、各種設定について紹介します

1. F5 DCS WAF について
====

F5 DCS WAFは、以下の特徴を備えております

- クラウドを含めたあらゆる環境に拡張、展開が可能
- F5 WAFで実装されているシグネチャを適応
- F5 Labが収集した実攻撃情報をもとに作成した高精度シグネチャ（Threat campaings）を無償バンドル
- 機械学習による誤検知抑制機能を実装
- シグネチャ検出以外の各種回避テクニックの検出にも対応
- Botシグネチャを標準実装。BotをGood/Suspicious/Maliciousに自動的に分類
- ユーザの異常な行動を分析、特定し、悪意のある攻撃をブロック

F5 DCS WAAPはこれらの高度なセキュリティをアプリケーションの迅速な展開に合わせて自由にご利用いただける環境を実現します

2. App Firewall の設定
====

1. App Firewall の設定項目
----

F5 DCS では、``App Firewall`` でWAFのセキュリティポリシーを管理することが可能です。

App Firewall で表示される主要な項目について説明します。実際の画面は次の新規作成の手順でご確認ください

- Enforcement Mode App: 
    Firewallで検知する脅威に対し、通信を遮断する(Blocking)か、遮断せず検知をする(Monitoring)を指定します
- Security Policy: 
    脅威に対する制御方法を指定します。以下の項目に関する制御を行います

  =================================== ========================================
  Signature Attack Type               攻撃手法を指定可能です(Command Execution等)
  Signature Selection By Accuracdy    Signatureの検知に対する精度を指定します
  Automatic Attack Signatures Tuning  Signatureの自動チューニングの利用有無を指定します
  Threat Campaings                    F5が提供するThreat Campaings機能の利用有無を指定します
  Violations                          どのような通信に対し違反行為を制御するか指定します
  =================================== ========================================

- Signature Based Bot Detection:
    Botの通信に対しどのように検知、制御するか指定します
- Allowed Response Status Codes:
    許可するHTTPレスポンスコードを指定します
- Mask Sensitive Parameters in Logs:
    ログ上で情報を秘匿化(Mask)するパラメータを指定します
- Blocking Response Page:
    App Firewallがブロックした際に応答するブロックページを指定します

2. App Firewall の作成
----

以下のメニューより、新規にセキュリティポリシーを作成します

画面左側 Security欄の ``App Firewall`` を開き、画面上部 ``Add App Firewall`` をクリックします

   .. image:: ./media/dcs-menu-app-fw.jpg
       :width: 400

設定内容は以下の通りです。表に示したパラメター以外の項目についても ``Custom`` を選択しておりますが、こちらは設定内容を表示する目的であり表示された各種詳細なパラメータの変更は行っておりません

-  入力パラメータ

   ================================= ==========================================================
   Name                              demo-app-fw
   --------------------------------- ----------------------------------------------------------
   Enforcement Mode                  Blocking
   --------------------------------- ----------------------------------------------------------
   Allowed Response Status Code      Custom
   --------------------------------- ----------------------------------------------------------
    + List of Response code          200
   --------------------------------- ----------------------------------------------------------
   Mask Sensitive Parameters in Logs Custom
   --------------------------------- ----------------------------------------------------------
    + Configuration                  ``Add Item`` をクリックし、Query Parameter / mypass を指定
   --------------------------------- ----------------------------------------------------------
   Blocking Response Page            Custom
   --------------------------------- ----------------------------------------------------------
    + Custom Blocking Page Body      Request Rejected の後ろに ``Custom Page`` を追加
   ================================= ==========================================================

   .. image:: ./media/dcs-app-fw.jpg
       :width: 400


3. HTTP Load Balancer で App Firewall Policy の指定
----

作成済みのHTTP Load Balancerに作成した App Firewall Policyを割り当てます
HTTP Load Balancer の設定手順は `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module3/module3.html>`__ を参照ください


画面左側 Manage欄の ``Load Balancers`` 、 ``HTTP Load Balancers`` を開き、対象のLoad Balancerを表示し画面右側に遷移します。

   .. image:: ./media/dcs-edit-lb.jpg
       :width: 400

すでに作成済みのオブジェクトを変更する場合、対象のオブジェクト一番右側 ``‥`` から、 ``Manage Configuration`` をクリックします

   .. image:: ./media/dcs-edit-lb2.jpg
       :width: 400

設定の結果が一覧で表示されます。画面右上 ``Edit Configuration`` から設定の変更します。 
Security COnfiguration 欄の ``Select Web Application Firewall (WAF) Config`` で ``App Firewall`` を選択し、
作成したApp Firewallのポリシーを選択してください。

   .. image:: ./media/dcs-edit-lb3.jpg
       :width: 400

2. 動作確認
====

Curlコマンドを使って各リクエストを送信し、その結果を確認します。リクエストを送信してから、ログの反映には1～2分ほどかかる場合があります。

.. NOTE::
  Curlコマンドを使用する環境でhostsファイルの変更が難しい場合、``--resolve`` オプションを指定し、リクエストの送信が可能です

  | # 今回のテストを想定したサンプルコマンド
  | curl -k -v --resolve echoapp.f5demo.net:443:<IP Address> https://echoapp.f5demo.net

各リクエストのログは以下の手順で参照することが可能です

   .. image:: ./media/dcs-app-fw-log.jpg
       :width: 400

   .. image:: ./media/dcs-app-fw-log2.jpg
       :width: 400

1. 正常動作
----

Curlコマンドで ``https://echoapp.f5demo.net`` へリクエストを送信し、応答が正常であることを確認します

.. code-block:: bash
  :linenos:
  :caption: https://echoapp.f5demo.net への接続結果
  :emphasize-lines: 12,16

  $ curl -k -v https://echoapp.f5demo.net
  
  ** 省略 **

  > GET / HTTP/2
  > Host: echoapp.f5demo.net
  > User-Agent: curl/7.58.0
  > Accept: */*

  ** 省略 **

  < HTTP/2 200
  < content-type: application/json
  < content-length: 735
  
  {"request":{"headers":[["host","app1.test10demo.xyz"],["user-agent","curl/7.58.0"],["accept","*/*"],["x-forwarded-for","18.178.83.1"],["x-forwarded-proto","https"],["x-envoy-external-address","18.178.83.1"],["x-request-id","91097bfc-7f80-487f-a028-014f9fab330e"],["content-length","0"]],"status":0,"httpversion":"1.1","method":"GET","scheme":"https","uri":"/","requestText":"","fullPath":"/"},"network":{"clientPort":"51117","clientAddress":"103.135.56.116","serverAddress":"172.21.0.2","serverPort":"443"},"ssl":{"isHttps":true,"sslProtocol":"TLSv1.2","sslCipher":"ECDHE-ECDSA-AES128-GCM-SHA256"},"session":{"requestId":"ccab5c27dd0fea280c42d4e447eaee54","connection":"20","connectionNumber":"1"},"environment":{"hostname":"echoapp"}}u

Response Code 200 が応答され、正しくコンテンツが表示されていることが確認できます。

このリクエストの結果は以下の通りです

- Security Event 画面の結果

   .. image:: ./media/dcs-app-fw-log-permit.jpg
       :width: 600

.. code-block:: json
  :linenos:
  :caption: https://echoapp.f5demo.net への接続結果を示すWAF Event
  :emphasize-lines: 4,25,46,69,71

  {
    "app_type": "",
    "signatures": {},
    "req_id": "91097bfc-7f80-487f-a028-014f9fab330e",
    "hostname": "master-0",
    "bot_verification_failed": false,
    "original_authority": "",
    "rtt_upstream_seconds": "",
    "src_instance": "JP",
    "req_headers": "{\"Accept\":\"*/*\",\"Host\":\"echoapp.f5demo.net\",\"Method\":\"GET\",\"Path\":\"/\",\"Scheme\":\"https\",\"User-Agent\":\"curl/7.58.0\",\"X-Envoy-External-Address\":\"18.178.83.1\",\"X-Forwarded-For\":\"18.178.83.1\",\"X-Forwarded-Proto\":\"https\",\"X-Request-Id\":\"91097bfc-7f80-487f-a028-014f9fab330e\"}",
    "tenant": "f5-apac-ent-uppdoshj",
    "app": "obelix",
    "policy_hits": {
      "policy_hits": {}
    },
    "method": "GET",
    "threat_campaigns": {},
    "violations": {},
    "source_type": "kafka",
    "dst_instance": "",
    "x_forwarded_for": "18.178.83.1",
    "duration_with_no_data_tx_delay": "",
    "waf_rule_tags": "{}",
    "rsp_code_class": "",
    "waf_mode": "allow",
    "time_to_last_upstream_rx_byte": 0,
    "scheme": "",
    "city": "Tokyo",
    "dst_site": "",
    "latitude": "35.689300",
    "messageid": "c102667e-dea5-4551-b495-71bf4217a9f6",
    "no_active_detections": false,
    "tls_version": "",
    "duration_with_data_tx_delay": "",
    "stream": "svcfw",
    "violation_rating": "0",
    "req_size": "208",
    "waf_rules_hit": "[]",
    "tls_fingerprint": "456523fc94726331a4d5a2e1d40b2cd7",
    "bot_name": "curl",
    "time_to_first_upstream_rx_byte": 0,
    "sni": "echoapp.f5demo.net",
    "response_flags": "",
    "site": "ty8-tky",
    "@timestamp": "2022-02-24T15:38:01.123Z",
    "calculated_action": "report",
    "req_params": "",
    "sample_rate": "",
    "original_headers": [
      "method",
      "path",
      "scheme",
      "host",
      "user-agent",
      "accept",
      "x-forwarded-for",
      "x-forwarded-proto",
      "x-envoy-external-address",
      "x-request-id"
    ],
    "dst_port": "0",
    "req_path": "/",
    "asn": "AMAZON-02(16509)",
    "node_id": "",
    "proxy_type": "",
    "is_truncated_field": false,
    "country": "JP",
    "kubernetes": {},
    "browser_type": "curl",
    "device_type": "Other",
    "bot_classification": "suspicious",
    "vhost_id": "6c0bb878-7ecb-4b20-815e-1f3521b12ff4",
    "detections": {},
    "longitude": "139.689900",
    "rtt_downstream_seconds": "",
    "http_version": "HTTP/1.1",
    "time_to_last_downstream_tx_byte": 0,
    "waf_rule_hit_count": "",
    "num_rules_hit": "",
    "vh_type": "",
    "rsp_size": "921",
    "api_endpoint": "{}",
    "authority": "echoapp.f5demo.net",
    "region": "13",
    "time_to_first_downstream_tx_byte": 0,
    "rsp_code_details": "",
    "dst": "",
    "connection_state": "",
    "dst_ip": "72.19.3.189",
    "is_new_dcid": true,
    "network": "18.176.0.0",
    "src_site": "ty8-tky",
    "src_ip": "18.178.83.1",
    "tls_cipher_suite": "",
    "bot_type": "HTTP Library",
    "original_path": "",
    "message_key": null,
    "user_agent": "curl/7.58.0",
    "severity": "info",
    "cluster_name": "ty8-tky-int-ves-io",
    "headers": {},
    "types": "input:string",
    "src": "N:public",
    "rsp_code": "200",
    "time_to_first_upstream_tx_byte": 0,
    "attack_types": {},
    "src_port": "40472",
    "dcid": "1645717081123-777275537",
    "req_body": "",
    "time_to_last_upstream_tx_byte": 0,
    "namespace": "h-matsumoto",
    "time": "2022-02-24T15:38:01.123Z",
    "waf_instance_id": "",
    "sec_event_type": "waf_sec_event",
    "user": "IP-18.178.83.1",
    "vh_name": "ves-io-http-loadbalancer-demo-echo-lb"
  }

- 4行目 ``req_id`` はそのログメッセージを特定するためのIDです。本サンプルリクエストでは通信がブロックされていないため、通信の応答として情報は表示されませんが、通信がブロックされた場合には ``support ID`` としてこの情報が表示されます
- 25行目 ``waf_mode`` が許可( ``Allow`` )、46行目 ``calculated_action`` が 通知( ``report`` ) であると確認できます
- 69行目 ``browser_type`` で ``curl`` と判定され、71行目 ``bot_classification`` で ``suspicious`` であると確認できます。これはCurlコマンドであることをBot Signatureの機能により判定しておりますが、suspiciousの設定に従って ``Report`` と処理し、拒否は行っておりません

この他にも様々な情報が表示されており、Security Eventから通信の詳細について把握することが可能となっています


2. Signatureによる攻撃の検知
----

Curlコマンドで ``https://echoapp.f5demo.net?a=<script>`` へリクエストを送信し、通信が ``ブロック`` されることを確認します

.. code-block:: bash
  :linenos:
  :caption: https://echoapp.f5demo.net?a=<script> への接続結果
  :emphasize-lines:  19

  $ curl -k -v "https://echoapp.f5demo.net?a=<script>"

  ** 省略 **

  > GET /?a=<script> HTTP/2
  > Host: echoapp.f5demo.net
  > User-Agent: curl/7.58.0
  > Accept: */*

  ** 省略 **

  < HTTP/2 200
  < content-length: 278
  < content-type: text/html; charset=UTF-8

  ** 省略 **

  * Connection #0 to host echoapp.f5demo.net left intact
  <html><head><title>Request Rejected Custom Page</title></head><body>The requested URL was rejected. Please consult with your administrator.<br/><br/>Your support ID is: 4813018f-1d4b-41e4-9284-144aadbbf578<br/><br/><a href="javascript:history.back()">

| この例では、URL ParameterにXSSに該当する文字列( ``<script>`` )が含まれているため、ポリシーでブロックされていることがわかります。
| ブロックページは、titleが、 ``Request Rejected Custom Page`` となっており、Custom Pageで指定した内容が反映されていることが確認できます。
| Support IDを見ると、 ``4813018f-1d4b-41e4-9284-144aadbbf578`` という値が記載されています

それではログを確認しましょう

- Security Event 画面の結果

   .. image:: ./media/dcs-app-fw-log-sig.jpg
       :width: 600

.. code-block:: json
  :linenos:
  :caption: https://echoapp.f5demo.net?a=<script> への接続結果を示すWAF Event
  :emphasize-lines: 3-44,45,66,77,87,147-151

  {
    "app_type": "",
    "signatures": [
      {
        "attack_type": "ATTACK_TYPE_CROSS_SITE_SCRIPTING",
        "matching_info": "Matched 7 characters on offset 24 against value: 'method: GET\r\npath: /?a=<script>\r\nscheme: https\r\nhost: echoapp.f'. ",
        "context": "header (path)",
        "name": "XSS script tag end (Headers)",
        "accuracy": "high_accuracy",
        "id": "200000091",
        "state": "Enabled",
        "id_name": "200000091, XSS script tag end (Headers)"
      },
      {
        "attack_type": "ATTACK_TYPE_CROSS_SITE_SCRIPTING",
        "matching_info": "Matched 7 characters on offset 23 against value: 'method: GET\r\npath: /?a=<script>\r\nscheme: https\r\nhost: echoapp.f'. ",
        "context": "header (path)",
        "name": "XSS script tag (Headers)",
        "accuracy": "high_accuracy",
        "id": "200000097",
        "state": "Enabled",
        "id_name": "200000097, XSS script tag (Headers)"
      },
      {
        "attack_type": "ATTACK_TYPE_CROSS_SITE_SCRIPTING",
        "matching_info": "Matched 7 characters on offset 2 against value: 'a=<script>'. ",
        "context": "parameter (a)",
        "name": "XSS script tag (Parameter)",
        "accuracy": "high_accuracy",
        "id": "200000098",
        "state": "Enabled",
        "id_name": "200000098, XSS script tag (Parameter)"
      },
      {
        "attack_type": "ATTACK_TYPE_CROSS_SITE_SCRIPTING",
        "matching_info": "Matched 7 characters on offset 3 against value: 'a=<script>'. ",
        "context": "parameter (a)",
        "name": "XSS script tag end (Parameter) (2)",
        "accuracy": "high_accuracy",
        "id": "200001475",
        "state": "Enabled",
        "id_name": "200001475, XSS script tag end (Parameter) (2)"
      }
    ],
    "req_id": "4813018f-1d4b-41e4-9284-144aadbbf578",
    "hostname": "master-2",
    "bot_verification_failed": false,
    "original_authority": "",
    "rtt_upstream_seconds": "",
    "src_instance": "JP",
    "req_headers": "{\"Accept\":\"*/*\",\"Host\":\"echoapp.f5demo.net\",\"Method\":\"GET\",\"Path\":\"/?a=\\u003cscript\\u003e\",\"Scheme\":\"https\",\"User-Agent\":\"curl/7.58.0\",\"X-Envoy-External-Address\":\"18.178.83.1\",\"X-Forwarded-For\":\"18.178.83.1\",\"X-Forwarded-Proto\":\"https\",\"X-Request-Id\":\"4813018f-1d4b-41e4-9284-144aadbbf578\"}",
    "tenant": "f5-apac-ent-uppdoshj",
    "app": "obelix",
    "policy_hits": {
      "policy_hits": {}
    },
    "method": "GET",
    "threat_campaigns": {},
    "violations": {},
    "source_type": "kafka",
    "dst_instance": "",
    "x_forwarded_for": "18.178.83.1",
    "duration_with_no_data_tx_delay": "",
    "waf_rule_tags": "{}",
    "rsp_code_class": "2xx",
    "waf_mode": "block",
    "time_to_last_upstream_rx_byte": 0,
    "scheme": "",
    "city": "Tokyo",
    "dst_site": "",
    "latitude": "35.689300",
    "messageid": "c102667e-dea5-4551-b495-71bf4217a9f6",
    "no_active_detections": false,
    "tls_version": "",
    "duration_with_data_tx_delay": "",
    "stream": "svcfw",
    "violation_rating": "5",
    "req_size": "219",
    "waf_rules_hit": "[]",
    "tls_fingerprint": "456523fc94726331a4d5a2e1d40b2cd7",
    "bot_name": "curl",
    "time_to_first_upstream_rx_byte": 0,
    "sni": "echoapp.f5demo.net",
    "response_flags": "",
    "site": "ty8-tky",
    "@timestamp": "2022-02-24T15:40:47.470Z",
    "calculated_action": "block",
    "req_params": "a=<script>",
    "sample_rate": "",
    "original_headers": [
      "method",
      "path",
      "scheme",
      "host",
      "user-agent",
      "accept",
      "x-forwarded-for",
      "x-forwarded-proto",
      "x-envoy-external-address",
      "x-request-id"
    ],
    "dst_port": "0",
    "req_path": "/",
    "asn": "AMAZON-02(16509)",
    "node_id": "",
    "proxy_type": "",
    "is_truncated_field": false,
    "country": "JP",
    "kubernetes": {},
    "browser_type": "curl",
    "device_type": "Other",
    "bot_classification": "suspicious",
    "vhost_id": "6c0bb878-7ecb-4b20-815e-1f3521b12ff4",
    "detections": {},
    "longitude": "139.689900",
    "rtt_downstream_seconds": "",
    "http_version": "HTTP/1.1",
    "time_to_last_downstream_tx_byte": 0,
    "waf_rule_hit_count": "",
    "num_rules_hit": "",
    "vh_type": "",
    "rsp_size": "0",
    "api_endpoint": "{}",
    "authority": "echoapp.f5demo.net",
    "region": "13",
    "time_to_first_downstream_tx_byte": 0,
    "rsp_code_details": "",
    "dst": "",
    "connection_state": "",
    "dst_ip": "72.19.3.189",
    "is_new_dcid": true,
    "network": "18.176.0.0",
    "src_site": "ty8-tky",
    "src_ip": "18.178.83.1",
    "tls_cipher_suite": "",
    "bot_type": "HTTP Library",
    "original_path": "",
    "message_key": null,
    "user_agent": "curl/7.58.0",
    "severity": "info",
    "cluster_name": "ty8-tky-int-ves-io",
    "headers": {},
    "types": "input:string",
    "src": "N:public",
    "rsp_code": "200",
    "time_to_first_upstream_tx_byte": 0,
    "attack_types": [
      {
        "name": "ATTACK_TYPE_CROSS_SITE_SCRIPTING"
      }
    ],
    "src_port": "40478",
    "dcid": "1645717247469-890683506",
    "req_body": "",
    "time_to_last_upstream_tx_byte": 0,
    "namespace": "h-matsumoto",
    "time": "2022-02-24T15:40:47.470Z",
    "waf_instance_id": "",
    "sec_event_type": "waf_sec_event",
    "user": "IP-18.178.83.1",
    "vh_name": "ves-io-http-loadbalancer-demo-echo-lb"
  }


- 66行目 ``waf_mode`` が拒否( ``Block`` )、87行目 ``calculated_action`` が 拒否( ``block`` ) となり通信が拒否されていることが確認できます
- 45行目 ``req_id`` は ブロックページ に表示された ``Support ID`` の値 ``4813018f-1d4b-41e4-9284-144aadbbf578`` であることが確認できます
- 3行目 から 44行目に表示されている内容が該当するSignatureを示します。内容を確認すると Cross Site Scripting(XSS)の攻撃であると検知していることが確認できます
- 77行目 ``violation_rating`` が ``5`` となっており、高い値となっております
- 147行目 から 151行目 ``attack_types`` で ``ATTACK_TYPE_CROSS_SITE_SCRIPTING`` と表示されており、XSSと検知されていることが確認できます

このように、ブロックページに表示されたSupport IDから対象のログを特定し、どのような理由により通信がブロックされたか確認することが可能です


3. Sensitive Dataのマスキング
----

Curlコマンドで ``https://echoapp.f5demo.net?mypass=secret`` へリクエストを送信し、通信が ``ブロック`` されることを確認します

.. code-block:: bash
  :linenos:
  :caption: https://echoapp.f5demo.net?mypass=secret への接続結果
  :emphasize-lines:  19

  $ curl -k -v https://echoapp.f5demo.net?mypass=secret

  ** 省略 **

  > GET /?mypass=secret HTTP/2
  > Host: echoapp.f5demo.net
  > User-Agent: curl/7.58.0
  > Accept: */*

  ** 省略 **

  < HTTP/2 200
  < content-type: application/json
  < content-length: 775

  ** 省略 **

  {"request":{"headers":[["host","app2.test10demo.xyz"],["user-agent","curl/7.58.0"],["accept","*/*"],["x-forwarded-for","18.178.83.1"],["x-forwarded-proto","https"],["x-envoy-external-address","18.178.83.1"],["x-request-id","22032402-0f75-412e-a1ac-c8c2afdb6ba7"],["content-length","0"]],"status":0,"httpversion":"1.1","method":"GET","scheme":"https","uri":"/","args":{"mypass":"secret"},"requestText":"","fullPath":"/?mypass=secret"},"network":{"clientPort":"33274","clientAddress":"103.135.56.97","serverAddress":"172.21.0.2","serverPort":"443"},"ssl":{"isHttps":true,"sslProtocol":"TLSv1.2","sslCipher":"ECDHE-ECDSA-AES128-GCM-SHA256"},"session":{"requestId":"abea7d90b1fb3ae939ccde985b149e05","connection":"21","connectionNumber":"1"},"environment":{"hostname":"echoapp"}}

この例では、通信はブロックされず正しく応答されていることが確認できます。
ポリシーではsensitive-parameterを指定しており、 ``mypass`` がURL Parameterに含まれる場合、その値をLOG上でマスクするよう設定しました。

それではログを確認しましょう

- Security Event 画面の結果

   .. image:: ./media/dcs-app-fw-log-sensitive-data.jpg
       :width: 600

.. code-block:: json
  :linenos:
  :caption: https://echoapp.f5demo.net?mypass=secret への接続結果を示すWAF Event
  :emphasize-lines: 4,25,46,47,16

  {
    "app_type": "",
    "signatures": {},
    "req_id": "22032402-0f75-412e-a1ac-c8c2afdb6ba7",
    "hostname": "master-2",
    "bot_verification_failed": false,
    "original_authority": "",
    "rtt_upstream_seconds": "",
    "src_instance": "JP",
    "req_headers": "{\"Accept\":\"*/*\",\"Host\":\"echoapp.f5demo.net\",\"Method\":\"GET\",\"Path\":\"/?mypass=******\",\"Scheme\":\"https\",\"User-Agent\":\"curl/7.58.0\",\"X-Envoy-External-Address\":\"18.178.83.1\",\"X-Forwarded-For\":\"18.178.83.1\",\"X-Forwarded-Proto\":\"https\",\"X-Request-Id\":\"22032402-0f75-412e-a1ac-c8c2afdb6ba7\"}",
    "tenant": "f5-apac-ent-uppdoshj",
    "app": "obelix",
    "policy_hits": {
      "policy_hits": {}
    },
    "method": "GET",
    "threat_campaigns": {},
    "violations": {},
    "source_type": "kafka",
    "dst_instance": "",
    "x_forwarded_for": "18.178.83.1",
    "duration_with_no_data_tx_delay": "",
    "waf_rule_tags": "{}",
    "rsp_code_class": "",
    "waf_mode": "allow",
    "time_to_last_upstream_rx_byte": 0,
    "scheme": "",
    "city": "Tokyo",
    "dst_site": "",
    "latitude": "35.689300",
    "messageid": "c102667e-dea5-4551-b495-71bf4217a9f6",
    "no_active_detections": false,
    "tls_version": "",
    "duration_with_data_tx_delay": "",
    "stream": "svcfw",
    "violation_rating": "0",
    "req_size": "222",
    "waf_rules_hit": "[]",
    "tls_fingerprint": "456523fc94726331a4d5a2e1d40b2cd7",
    "bot_name": "curl",
    "time_to_first_upstream_rx_byte": 0,
    "sni": "echoapp.f5demo.net",
    "response_flags": "",
    "site": "ty8-tky",
    "@timestamp": "2022-02-24T15:41:43.531Z",
    "calculated_action": "report",
    "req_params": "mypass=******",
    "sample_rate": "",
    "original_headers": [
      "method",
      "path",
      "scheme",
      "host",
      "user-agent",
      "accept",
      "x-forwarded-for",
      "x-forwarded-proto",
      "x-envoy-external-address",
      "x-request-id"
    ],
    "dst_port": "0",
    "req_path": "/",
    "asn": "AMAZON-02(16509)",
    "node_id": "",
    "proxy_type": "",
    "is_truncated_field": false,
    "country": "JP",
    "kubernetes": {},
    "browser_type": "curl",
    "device_type": "Other",
    "bot_classification": "suspicious",
    "vhost_id": "6c0bb878-7ecb-4b20-815e-1f3521b12ff4",
    "detections": {},
    "longitude": "139.689900",
    "rtt_downstream_seconds": "",
    "http_version": "HTTP/1.1",
    "time_to_last_downstream_tx_byte": 0,
    "waf_rule_hit_count": "",
    "num_rules_hit": "",
    "vh_type": "",
    "rsp_size": "961",
    "api_endpoint": "{}",
    "authority": "echoapp.f5demo.net",
    "region": "13",
    "time_to_first_downstream_tx_byte": 0,
    "rsp_code_details": "",
    "dst": "",
    "connection_state": "",
    "dst_ip": "72.19.3.189",
    "is_new_dcid": true,
    "network": "18.176.0.0",
    "src_site": "ty8-tky",
    "src_ip": "18.178.83.1",
    "tls_cipher_suite": "",
    "bot_type": "HTTP Library",
    "original_path": "",
    "message_key": null,
    "user_agent": "curl/7.58.0",
    "severity": "info",
    "cluster_name": "ty8-tky-int-ves-io",
    "headers": {},
    "types": "input:string",
    "src": "N:public",
    "rsp_code": "200",
    "time_to_first_upstream_tx_byte": 0,
    "attack_types": {},
    "src_port": "40480",
    "dcid": "1645717303530-100012152",
    "req_body": "",
    "time_to_last_upstream_tx_byte": 0,
    "namespace": "h-matsumoto",
    "time": "2022-02-24T15:41:43.531Z",
    "waf_instance_id": "",
    "sec_event_type": "waf_sec_event",
    "user": "IP-18.178.83.1",
    "vh_name": "ves-io-http-loadbalancer-demo-echo-lb"
  }

- 4行目 ``req_id`` はそのログメッセージを特定するためのIDです。本サンプルリクエストでは通信がブロックされていないため、通信の応答として情報は表示されませんが、通信がブロックされた場合には ``support ID`` としてこの情報が表示されます
- 25行目 ``waf_mode`` が許可( ``Allow`` )、46行目 ``calculated_action`` が 通知( ``report`` ) であると確認できます
- 47行目 でリクエストのQuery Parameterが表示されており、 ``req_params`` の値が ``mypass=******`` となっています。これは ``Mask Sensitive Parameters`` の設定により指定したパラメータが Query Parameter に含まれるため、その値を Sensitive Data として扱い、ログ上でMaskしています。さらに、10行目の ``req_headers`` にもこの情報が含まれておりMaskされていることが確認できます


4. Originから503が応答される場合の動作
----

Curlコマンドで ``https://echoapp.f5demo.net/503`` へリクエストを送信し、通信が ``ブロック`` されることを確認します

.. code-block:: bash
  :linenos:
  :caption: https://echoapp.f5demo.net/503 への接続結果
  :emphasize-lines:  18

  $ curl -k -v https://echoapp.f5demo.net/503

  ** 省略 **

  > GET /503 HTTP/2
  > Host: echoapp.f5demo.net
  > User-Agent: curl/7.58.0
  > Accept: */*

  ** 省略 **

  < HTTP/2 200
  < content-type: text/html; charset=UTF-8
  < content-length: 278

  ** 省略 **

  <html><head><title>Request Rejected Custom Page</title></head><body>The requested URL was rejected. Please consult with your administrator.<br/><br/>Your support ID is: bf5e1262-fe22-46f6-9661-664c46d6ca16<br/><br/><a href="javascript:history.back()">[Go Back]</a></body></html>

サンプルアプリケーションでは、 ``/503`` にアクセスすると、 HTTP Response Code 503 が応答される動作となります。
応答の結果を確認すると通信がブロックされています。

それではログを確認しましょう

- Security Event 画面の結果

   .. image:: ./media/dcs-app-fw-log-response-code.jpg
       :width: 600

.. code-block:: json
  :linenos:
  :caption: https://echoapp.f5demo.net/503 への接続結果を示すWAF Event
  :emphasize-lines: 4,25,46

  {
    "app_type": "",
    "signatures": {},
    "req_id": "bf5e1262-fe22-46f6-9661-664c46d6ca16",
    "hostname": "master-1",
    "bot_verification_failed": false,
    "original_authority": "",
    "rtt_upstream_seconds": "",
    "src_instance": "JP",
    "req_headers": "{\"Accept\":\"*/*\",\"Host\":\"echoapp.f5demo.net\",\"Method\":\"GET\",\"Path\":\"/503\",\"Scheme\":\"https\",\"User-Agent\":\"curl/7.58.0\",\"X-Envoy-External-Address\":\"18.178.83.1\",\"X-Forwarded-For\":\"18.178.83.1\",\"X-Forwarded-Proto\":\"https\",\"X-Request-Id\":\"bf5e1262-fe22-46f6-9661-664c46d6ca16\"}",
    "tenant": "f5-apac-ent-uppdoshj",
    "app": "obelix",
    "policy_hits": {
      "policy_hits": {}
    },
    "method": "GET",
    "threat_campaigns": {},
    "violations": {},
    "source_type": "kafka",
    "dst_instance": "",
    "x_forwarded_for": "18.178.83.1",
    "duration_with_no_data_tx_delay": "",
    "waf_rule_tags": "{}",
    "rsp_code_class": "",
    "waf_mode": "allow",
    "time_to_last_upstream_rx_byte": 0,
    "scheme": "",
    "city": "Tokyo",
    "dst_site": "",
    "latitude": "35.689300",
    "messageid": "c102667e-dea5-4551-b495-71bf4217a9f6",
    "no_active_detections": false,
    "tls_version": "",
    "duration_with_data_tx_delay": "",
    "stream": "svcfw",
    "violation_rating": "0",
    "req_size": "211",
    "waf_rules_hit": "[]",
    "tls_fingerprint": "456523fc94726331a4d5a2e1d40b2cd7",
    "bot_name": "curl",
    "time_to_first_upstream_rx_byte": 0,
    "sni": "echoapp.f5demo.net",
    "response_flags": "",
    "site": "ty8-tky",
    "@timestamp": "2022-02-24T15:44:48.969Z",
    "calculated_action": "report",
    "req_params": "",
    "sample_rate": "",
    "original_headers": [
      "method",
      "path",
      "scheme",
      "host",
      "user-agent",
      "accept",
      "x-forwarded-for",
      "x-forwarded-proto",
      "x-envoy-external-address",
      "x-request-id"
    ],
    "dst_port": "0",
    "req_path": "/503",
    "asn": "AMAZON-02(16509)",
    "node_id": "",
    "proxy_type": "",
    "is_truncated_field": false,
    "country": "JP",
    "kubernetes": {},
    "browser_type": "curl",
    "device_type": "Other",
    "bot_classification": "suspicious",
    "vhost_id": "6c0bb878-7ecb-4b20-815e-1f3521b12ff4",
    "detections": {},
    "longitude": "139.689900",
    "rtt_downstream_seconds": "",
    "http_version": "HTTP/1.1",
    "time_to_last_downstream_tx_byte": 0,
    "waf_rule_hit_count": "",
    "num_rules_hit": "",
    "vh_type": "",
    "rsp_size": "198",
    "api_endpoint": "{}",
    "authority": "echoapp.f5demo.net",
    "region": "13",
    "time_to_first_downstream_tx_byte": 0,
    "rsp_code_details": "",
    "dst": "",
    "connection_state": "",
    "dst_ip": "72.19.3.189",
    "is_new_dcid": true,
    "network": "18.176.0.0",
    "src_site": "ty8-tky",
    "src_ip": "18.178.83.1",
    "tls_cipher_suite": "",
    "bot_type": "HTTP Library",
    "original_path": "",
    "message_key": null,
    "user_agent": "curl/7.58.0",
    "severity": "info",
    "cluster_name": "ty8-tky-int-ves-io",
    "headers": {},
    "types": "input:string",
    "src": "N:public",
    "rsp_code": "200",
    "time_to_first_upstream_tx_byte": 0,
    "attack_types": {},
    "src_port": "40482",
    "dcid": "1645717488969-591222023",
    "req_body": "",
    "time_to_last_upstream_tx_byte": 0,
    "namespace": "h-matsumoto",
    "time": "2022-02-24T15:44:48.969Z",
    "waf_instance_id": "",
    "sec_event_type": "waf_sec_event",
    "user": "IP-18.178.83.1",
    "vh_name": "ves-io-http-loadbalancer-demo-echo-lb"
  }

- 4行目 ``req_id`` は ブロックページ に表示された ``Support ID`` の値 ``bf5e1262-fe22-46f6-9661-664c46d6ca16`` であることが確認できます
- しかし、25行目 ``waf_mode`` が許可( ``Allow`` )、46行目 ``calculated_action`` が 通知( ``report`` ) となり、拒否となっていないことが確認できます。この点がWAF Eventsのログと一致しません

もう一つログを確認します。対象のWAF Eventsと合わせてL7 Eventsが記録されているかとおもます。そちらを確認してください


   .. image:: ./media/dcs-app-fw-log-response-code2.jpg
       :width: 600

.. code-block:: json
  :linenos:
  :caption: https://echoapp.f5demo.net/503 への接続結果を示すL7 Event
  :emphasize-lines: 8,9,33-37

  {
    "country": "JP",
    "kubernetes": {},
    "l7_policy_rules_hit": "",
    "app_type": "h-matsumoto",
    "browser_type": "curl",
    "device_type": "Other",
    "req_id": "bf5e1262-fe22-46f6-9661-664c46d6ca16",
    "waf_action": "block",
    "hostname": "master-1",
    "original_authority": "app2.test10demo.xyz",
    "rtt_upstream_seconds": "0.014000",
    "src_instance": "JP",
    "req_headers": "null",
    "tenant": "f5-apac-ent-uppdoshj",
    "longitude": "139.689900",
    "app": "obelix",
    "rtt_downstream_seconds": "0.007000",
    "policy_hits": {
      "policy_hits": {}
    },
    "method": "GET",
    "time_to_last_downstream_tx_byte": 0.054213402,
    "waf_rule_hit_count": "0",
    "source_type": "kafka",
    "dst_instance": "18.178.83.1",
    "vh_type": "HTTP-LOAD-BALANCER",
    "x_forwarded_for": "18.178.83.1",
    "duration_with_no_data_tx_delay": "0.005670",
    "rsp_size": "802",
    "api_endpoint": "{\"collapsed_url\":\"UNKNOWN\",\"method\":\"GET\"}",
    "authority": "app2.test10demo.xyz",
    "app_firewall_info": {
      "name": "h-matsumoto:demo-app-fw",
      "action": "block",
      "description": "Disallowed response code (503)"
    },
    "region": "13",
    "time_to_first_downstream_tx_byte": 0.054180343,
    "rsp_code_class": "2xx",
    "rsp_code_details": "via_upstream",
    "time_to_last_upstream_rx_byte": 0.053070185,
    "dst": "S:app2.test10demo.xyz",
    "scheme": "https",
    "city": "Tokyo",
    "dst_site": "ty8-tky",
    "latitude": "35.689300",
    "messageid": "b5315f10-3181-4f8b-9c1e-3631817e22d6",
    "tls_version": "TLSv1_3",
    "connection_state": "CLOSED",
    "dst_ip": "NOT-APPLICABLE",
    "network": "18.176.0.0",
    "src_site": "ty8-tky",
    "terminated_time": "2022-02-24T15:44:48.970908229Z",
    "duration_with_data_tx_delay": "0.005703",
    "src_ip": "18.178.83.1",
    "connected_time": "2022-02-24T15:44:48.91520768Z",
    "stream": "svcfw",
    "tls_cipher_suite": "TLSv1_3/TLS_AES_256_GCM_SHA384",
    "original_path": "/503",
    "message_key": null,
    "req_size": "221",
    "user_agent": "curl/7.58.0",
    "severity": "info",
    "cluster_name": "ty8-tky-int-ves-io",
    "headers": {},
    "tls_fingerprint": "456523fc94726331a4d5a2e1d40b2cd7",
    "types": "input:string",
    "src": "N:public",
    "time_to_first_upstream_rx_byte": 0.0528934,
    "rsp_code": "200",
    "time_to_first_upstream_tx_byte": 0.048510615,
    "sni": "echoapp.f5demo.net",
    "response_flags": "",
    "src_port": "40482",
    "site": "ty8-tky",
    "@timestamp": "2022-02-24T15:44:49.614Z",
    "req_body": "",
    "req_params": "",
    "sample_rate": "1.000000",
    "time_to_last_upstream_tx_byte": 0.048521521,
    "dst_port": "443",
    "namespace": "h-matsumoto",
    "req_path": "/503",
    "time": "2022-02-24T15:44:49.614Z",
    "asn": "AMAZON-02(16509)",
    "sec_event_type": "l7_policy_sec_event",
    "user": "IP-18.178.83.1",
    "vh_name": "ves-io-http-loadbalancer-demo-echo-lb",
    "node_id": "envoy_1",
    "proxy_type": "http"
  }

- 8行目 ``req_id`` は ブロックページ に表示された ``Support ID`` の値 ``bf5e1262-fe22-46f6-9661-664c46d6ca16`` であることが確認できます
- 9行目 ``waf_action`` が拒否( ``block`` ) となっていることが確認できます
- 33行目 から 37行目 ``app_firewall_info`` の ``action`` と ``description`` を見ると、許可されないレスポンスコード( Disallowed response code (503) ) であるため拒否( ``block`` )されたことがわかります

このようにSecurity Eventsに表示されるログから通信がどのように制御されたものであるか確認することができます。


5. HTTP Protocol 違反の検知
----

プロトコル(Protocol)は予め通信の内容や仕組みが決まったものであり、通信はそれに則って行われます。
正常なクライアント・サーバはそのプロトコルの通りに動作しますが、攻撃者は本来のプロトコルの仕様に対して矛盾となる通信を行うことにより、アプリケーションの想定外な動作を引き起こす場合があります。

App Firewallでは ``Violation`` という仕組みにより、SignatureやBOTとはまた異なる、各種プロトコルの動作や悪意ある通信を検知・拒否することが可能です。
ここではシンプルな HTTP Protocol 違反を制御する動作を確認します

Curlコマンドで ``https://echoapp.f5demo.net/`` へリクエストを送信します。ただし、Protocolとして矛盾した動作となるため、以下のような情報でリクエストを送信します。

============ ===================
Method       POST
Content-Type application/json
送信データ    data=dummy
============ ===================

``Content-TYpe`` ではJSON形式( ``application/json`` )を指定していますが、 ``実際のデータ`` の矛盾により通信が ``ブロック`` されることを確認します

.. code-block:: bash
  :linenos:
  :caption: https://echoapp.f5demo.net/ への接続結果
  :emphasize-lines:  20

  $ curl -kv https://echoapp.f5demo.net/ -H "Content-Type: application/json" -X POST -d "data=dummy"
  
  ** 省略 **
  
  > POST / HTTP/2
  > Host: echoapp.f5demo.net
  > User-Agent: curl/7.58.0
  > Accept: */*
  > Content-Type: application/json
  > Content-Length: 10
  
  ** 省略 **
  
  < HTTP/2 200
  < content-length: 278
  < content-type: text/html; charset=UTF-8
  
  ** 省略 **
  
  <html><head><title>Request Rejected Custom Page</title></head><body>The requested URL was rejected. Please consult with your administrator.<br/><br/>Your support ID is: 5a253e51-b03a-465a-96b7-fa388298f759<br/><br/><a href="javascript:history.back()">[Go Back]</a></body></html>

応答の結果を確認すると通信がブロックされています。

それではログを確認しましょう

- Security Event 画面の結果

   .. image:: ./media/dcs-app-fw-log-violation.jpg
       :width: 600

.. code-block:: json
  :linenos:
  :caption: https://echoapp.f5demo.net/ への接続結果を示すWAF Event
  :emphasize-lines: 4,74,18-26,116-120

  {
    "app_type": "",
    "signatures": {},
    "req_id": "5a253e51-b03a-465a-96b7-fa388298f759",
    "hostname": "master-2",
    "bot_verification_failed": false,
    "original_authority": "",
    "rtt_upstream_seconds": "",
    "src_instance": "JP",
    "req_headers": "{\"Accept\":\"*/*\",\"Content-Length\":\"10\",\"Content-Type\":\"application/json\",\"Host\":\"echoapp.f5demo.net\",\"Method\":\"POST\",\"Path\":\"/\",\"Scheme\":\"https\",\"User-Agent\":\"curl/7.58.0\",\"X-Envoy-External-Address\":\"18.178.83.1\",\"X-Forwarded-For\":\"18.178.83.1\",\"X-Forwarded-Proto\":\"https\",\"X-Request-Id\":\"5a253e51-b03a-465a-96b7-fa388298f759\"}",
    "tenant": "f5-apac-ent-uppdoshj",
    "app": "obelix",
    "policy_hits": {
      "policy_hits": {}
    },
    "method": "POST",
    "threat_campaigns": {},
    "violations": [
      {
        "attack_type": "ATTACK_TYPE_JSON_PARSER_ATTACK",
        "matching_info": "",
        "context": "URL",
        "name": "VIOL_JSON_MALFORMED",
        "state": "Enabled"
      }
    ],
    "source_type": "kafka",
    "dst_instance": "",
    "x_forwarded_for": "18.178.83.1",
    "duration_with_no_data_tx_delay": "",
    "waf_rule_tags": "{}",
    "rsp_code_class": "2xx",
    "waf_mode": "block",
    "time_to_last_upstream_rx_byte": 0,
    "scheme": "",
    "city": "Tokyo",
    "dst_site": "",
    "latitude": "35.689300",
    "messageid": "c102667e-dea5-4551-b495-71bf4217a9f6",
    "no_active_detections": false,
    "tls_version": "",
    "duration_with_data_tx_delay": "",
    "stream": "svcfw",
    "violation_rating": "3",
    "req_size": "263",
    "waf_rules_hit": "[]",
    "tls_fingerprint": "456523fc94726331a4d5a2e1d40b2cd7",
    "bot_name": "curl",
    "time_to_first_upstream_rx_byte": 0,
    "sni": "echoapp.f5demo.net",
    "response_flags": "",
    "site": "ty8-tky",
    "@timestamp": "2022-02-25T04:08:03.197Z",
    "calculated_action": "block",
    "req_params": "",
    "sample_rate": "",
    "original_headers": [
      "method",
      "path",
      "scheme",
      "host",
      "user-agent",
      "accept",
      "content-type",
      "content-length",
      "x-forwarded-for",
      "x-forwarded-proto",
      "x-envoy-external-address",
      "x-request-id"
    ],
    "dst_port": "0",
    "req_path": "/",
    "asn": "AMAZON-02(16509)",
    "node_id": "",
    "proxy_type": "",
    "is_truncated_field": false,
    "country": "JP",
    "kubernetes": {},
    "browser_type": "curl",
    "device_type": "Other",
    "bot_classification": "suspicious",
    "vhost_id": "6c0bb878-7ecb-4b20-815e-1f3521b12ff4",
    "detections": {},
    "longitude": "139.689900",
    "rtt_downstream_seconds": "",
    "http_version": "HTTP/1.1",
    "time_to_last_downstream_tx_byte": 0,
    "waf_rule_hit_count": "",
    "num_rules_hit": "",
    "vh_type": "",
    "rsp_size": "0",
    "api_endpoint": "{}",
    "authority": "echoapp.f5demo.net",
    "region": "13",
    "time_to_first_downstream_tx_byte": 0,
    "rsp_code_details": "",
    "dst": "",
    "connection_state": "",
    "dst_ip": "72.19.3.189",
    "is_new_dcid": true,
    "network": "18.176.0.0",
    "src_site": "ty8-tky",
    "src_ip": "18.178.83.1",
    "tls_cipher_suite": "",
    "bot_type": "HTTP Library",
    "original_path": "",
    "message_key": null,
    "user_agent": "curl/7.58.0",
    "severity": "info",
    "cluster_name": "ty8-tky-int-ves-io",
    "headers": {},
    "types": "input:string",
    "src": "N:public",
    "rsp_code": "200",
    "time_to_first_upstream_tx_byte": 0,
    "attack_types": [
      {
        "name": "ATTACK_TYPE_JSON_PARSER_ATTACK"
      }
    ],
    "src_port": "40558",
    "dcid": "1645762083197-412728685",
    "req_body": "",
    "time_to_last_upstream_tx_byte": 0,
    "namespace": "h-matsumoto",
    "time": "2022-02-25T04:08:03.197Z",
    "waf_instance_id": "",
    "sec_event_type": "waf_sec_event",
    "user": "IP-18.178.83.1",
    "vh_name": "ves-io-http-loadbalancer-demo-echo-lb"
  }

- 66行目 ``waf_mode`` が拒否( ``block`` )、54行目 ``calculated_action`` が 拒否( ``block`` ) となり通信が拒否されていることが確認できます
- 4行目 ``req_id`` は ブロックページ に表示された ``Support ID`` の値 ``5a253e51-b03a-465a-96b7-fa388298f759`` であることが確認できます
- 18行目 から 26行目に表示されている内容が該当するViolationを示します。内容を確認すると ``ATTACK_TYPE_JSON_PARSER_ATTACK`` であり、正しいJSONの書式でない( ``VIOL_JSON_MALFORMED`` )と確認できます
- また、116行目から120行目 ``attack_types`` で ``ATTACK_TYPE_JSON_PARSER_ATTACK`` と表示されており、JSON PARSER ATTACKと検知されていることが確認できます


3. App Firewall Policyの解除
====

次の項目で、その他の機能を確認するための手順です。

`こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module5/module5.html#http-load-balancer-app-firewall-policy>`__ の手順を参考に、HTTP Load Balancerに割り当てたApp FirewallのPolicyを解除してください

   .. image:: ./media/dcs-app-fw-detach.jpg
       :width: 400

