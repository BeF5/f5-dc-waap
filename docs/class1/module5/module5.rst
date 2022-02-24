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

- Enforcement Mode App
  - Firewallで検知する脅威に対し、通信を遮断する(Blocking)か、遮断せず検知をする(Monitoring)を指定します
- Security Policy
  - 脅威に対する制御方法を指定します。以下の項目に関する制御を行います

  =================================== ========================================
  Signature Attack Type               攻撃手法を指定可能です(Command Execution等)
  Signature Selection By Accuracdy    Signatureの検知に対する精度を指定します
  Automatic Attack Signatures Tuning  Signatureの自動チューニングの利用有無を指定します
  Threat Campaings                    F5が提供するThreat Campaings機能の利用有無を指定します
  Violations                          どのような通信に対し違反行為を制御するか指定します
  =================================== ========================================

- Signature Based Bot Detection
  - Botの通信に対しどのように検知、制御するか指定します
- Allowed Response Status Codes
  - 許可するHTTPレスポンスコードを指定します
- Mask Sensitive Parameters in Logs
  - ログ上で情報を秘匿化(Mask)するパラメータを指定します
- Blocking Response Page
  - App Firewallがブロックした際に応答するブロックページを指定します

2. App Firewall の作成
----

以下のメニューより、新規にセキュリティポリシーを作成します

画面左側 Security欄の ``App Firewall`` を開き、画面上部 ``Add App Firewall`` をクリックします

設定内容は以下の通りです。表に示したパラメター以外の項目についても ``Custom`` を選択しておりますが、こちらは設定内容を表示する目的であり表示された各種詳細なパラメータの変更は行っておりません

-  入力パラメータ

   =========================== =============================
   Name                        demo-echo-lb
   --------------------------- -----------------------------
   List of Domain              echoapp.f5demo.net
   --------------------------- -----------------------------
   Select Type of Load Blancer HTTPS with Custom Certificate
   =========================== =============================

   .. image:: ./media/dcs-app-fw.jpg
       :width: 400


3. HTTP Load Balancer で App Firewall Policy の指定
----

2. 動作確認
====

Curlコマンドを使って各リクエストを送信し、その結果を確認します。リクエストを送信してから、ログの反映には1～2分ほどかかる場合があります。

.. NOTE::
  Curlコマンドを使用する環境でhostsファイルの変更が難しい場合、``--resolve`` オプションを指定し、リクエストの送信が可能です

  # 今回のテストを想定したサンプルコマンド
  curl -k -v --resolve echoapp.f5demo.net:443:<IP Address> https://echoapp.f5demo.net

各リクエストのログは以下の手順で参照することが可能です

   .. image:: ./media/dcs-app-fw.jpg
       :width: 400

1. 正常動作
----

Curlコマンドで ``https://echoapp.f5demo.net`` へリクエストを送信し、応答が正常であることを確認します

.. code-block:: bash
  :linenos:
  :caption: https://echoapp.f5demo.net への接続結果
  :emphasize-lines: 2

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

   .. image:: ./media/dcs-app-fw-log-permit.jpg
       :width: 400

.. code-block:: json
  :linenos:
  :caption: https://echoapp.f5demo.net への接続結果
  :emphasize-lines: 2

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

2. Signatureによる攻撃の検知
----

Curlコマンドで ``https://echoapp.f5demo.net?a=<script>`` へリクエストを送信し、通信が ``ブロック`` されることを確認します

.. code-block:: bash
  :linenos:
  :caption: https://echoapp.f5demo.net?a=<script> への接続結果
  :emphasize-lines: 3

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

| この例では、URL ParameterにXSSに該当する文字列が含まれているため、ポリシーでブロックされていることがわかります。
| ブロックページは、titleが、 ``Request Rejected Custom Page`` となっており、Custom Pageで指定した内容が反映されていることが確認できます。
| Support IDを見ると、``4813018f-1d4b-41e4-9284-144aadbbf578`` という値が記載されています。

それではログを確認しましょう

   .. image:: ./media/dcs-app-fw-log-sig.jpg
       :width: 400

.. code-block:: json
  :linenos:
  :caption: https://echoapp.f5demo.net?a=<script> への接続結果
  :emphasize-lines: 2


3. Sensitive Dataのマスキング
----

Curlコマンドで ``https://echoapp.f5demo.net?mypass=secret`` へリクエストを送信し、通信が ``ブロック`` されることを確認します

.. code-block:: bash
  :linenos:
  :caption: https://echoapp.f5demo.net への接続結果
  :emphasize-lines: 2

  $ curl -k -v https://echoapp.f5demo.net

  $ curl -k -v --resolve echoapp.f5demo.net:443:72.19.3.189 "https://echoapp.f5demo.net?mypass=secret"

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

   .. image:: ./media/dcs-app-fw-log-sensitive-data.jpg
       :width: 400

.. code-block:: json
  :linenos:
  :caption: https://echoapp.f5demo.net?mypass=secret> への接続結果
  :emphasize-lines: 2

4. Originから503が応答される場合の動作
----

Curlコマンドで ``https://echoapp.f5demo.net/503`` へリクエストを送信し、通信が ``ブロック`` されることを確認します

.. code-block:: bash
  :linenos:
  :caption: https://echoapp.f5demo.net への接続結果
  :emphasize-lines: 2

  $ curl -k -v https://echoapp.f5demo.net

  curl -k -v --resolve echoapp.f5demo.net:443:72.19.3.189 https://echoapp.f5demo.net/503

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

  <html><head><title>Request Rejected Custom Page</title></head><body>The requested URL was rejected. Please consult with your administrator.<br/><br/>Your support ID is: bf5e1262-fe22-46f6-9661-664c46d6ca16<br/><br/><a href="javascript:history.back()">[Go Back]</a></body></html>ubuntu@ip-10-0-11-227:~$

サンプルアプリケーションでは、 ``/503`` にアクセスすると、 HTTP Response Code 503 が応答される動作となります。
応答の結果を確認すると通信がブロックされています。

それではログを確認しましょう


   .. image:: ./media/dcs-app-fw-log-response-code.jpg
       :width: 400

.. code-block:: json
  :linenos:
  :caption: https://echoapp.f5demo.net/503 への接続結果
  :emphasize-lines: 2

