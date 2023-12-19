XC Rate Limit
####

※本資料の画面表示や名称は資料作成時点の画面表示を利用しております。アップデート等より表示が若干異なる場合がございます。

RateLimit を利用するための各種設定について紹介します

マニュアルは以下のページを参照してください

- `Configure Rate Limiting per User <https://docs.cloud.f5.com/docs/how-to/advanced-security/user-rate-limit>`__

1. Rate Limit について
====

RateLimit 対策機能により以下の操作をすることが可能です

- あるユーザに対し一定の期間に指定したリクエスト数を超える通信をブロックします
- ログイン、サインアップ画面などの重要なフローを保護し

XC WAAPはこれらの高度なセキュリティをアプリケーションの迅速な展開に合わせて自由にご利用いただける環境を実現します

2. RateLimit の設定
====

1. RateLimit の設定
----

作成済みのHTTP Load Balancerに RateLimit を設定します。
HTTP Load Balancer の設定手順は `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module03/module03.html>`__ を参照ください

画面左側 Manage欄の ``Load Balancers`` 、 ``HTTP Load Balancers`` を開き、対象のLoad Balancerを表示し画面右側に遷移します。

   .. image:: ../module06/media/dcs-edit-lb.jpg
       :width: 400

すでに作成済みのオブジェクトを変更する場合、対象のオブジェクト一番右側 ``‥`` から、 ``Manage Configuration`` をクリックします

   .. image:: ../module06/media/dcs-edit-lb2.jpg
       :width: 400

設定の結果が一覧で表示されます。画面右上 ``Edit Configuration`` から設定の変更します。 
Security Configuration 欄 右上の ``Show Advanced Fields`` をクリックします。

まず、 ``User Identifier`` が ``Rate Limiting Parameters`` となっていることを確認してください。
次に、 ``Rate Limiting`` で ``Rate limiting Parameters`` を選択し、 ``Configure`` をクリックしてください。
``Rate Limit Configuration`` が表示されますので、画面の内容が正しいことを確認し ``Apply`` をクリックしてください。

   .. image:: ./media/dcs-edit-lb-ratelimit.jpg
       :width: 400

正しく設定されたことを確認し、画面最下部の ``Apply`` をクリックしてください。

   .. image:: ./media/dcs-edit-lb-ratelimit2.jpg
       :width: 400




3. 動作確認
====

1. CurlコマンドによるRate Limitの確認
----

以下Curlコマンドを実行します。連続して2回のリクエストを送付するコマンドです

.. code-block:: bash
  :linenos:
  :caption: Curl コマンドを使った https://echoapp.f5demo.net への接続結果

  $ curl -vks https://echoapp.f5demo.net ; curl -vks https://echoapp.f5demo.net ;

  # 1回目のアクセスは正常に接続した結果が表示されます

  ** 省略 **

  > GET / HTTP/2
  > Host: echoapp.f5demo.net
  > User-Agent: curl/7.58.0

  ** 省略 **

  < HTTP/2 200
  < content-type: application/json
  
  ** 省略 **

  {"request":{"headers":[["host","app1.test10demo.xyz"],["user-agent","curl/7.58.0"],["accept","*/*"],["x-forwarded-for","18.178.83.1"],["x-forwarded-proto","https"],["x-envoy-external-address","18.178.83.1"],["x-request-id","c470fbb8-d762-496d-b8e1-a209a6410824"],["content-length","0"]],"status":0,"httpversion":"1.1","method":"GET","scheme":"http","uri":"/","requestText":"","fullPath":"/"},"network":{"clientPort":"57697","clientAddress":"103.135.56.116","serverAddress":"192.168.16.2","serverPort":"80"},"ssl":{"isHttps":false},"session":{"requestId":"0938eca4764809603c95fe4984c6fc4e","connection":"1445","connectionNumber":"1"},"environment":{"hostname":"echoapp"}}* Rebuilt URL to: https://echoapp.f5demo.net/
  
  # 2回目のアクセスは正常に接続した結果が表示されます
  
  ** 省略 **

  > GET / HTTP/2
  > Host: echoapp.f5demo.net
  > User-Agent: curl/7.58.0
  
  ** 省略 **

  < HTTP/2 429
  < content-type: text/html; charset=UTF-8

  ** 省略 **

  <!-- Body -->
  <div class="error-body">
    <h1>
    Error 429 - Too Many Requests
    </h1>

  ** 省略 **


| 1回目のアクセスは正しくOrigin Serverへ到達し、応答が返ってきていることが確認できます。
| 2回目のアクセスは、Rate Limitに該当し、Status Code 429が応答されており、エラーページのHTMLが応答されていることが確認できます。

2. ブラウザによるRate Limitの確認
----

あるクライアントから短い時間で複数のアクセスがあった場合ブロックされることが確認できました。
ブラウザで ``https://echoapp.f5demo.net`` にアクセスし、ページを閲覧してください。

ページを複数回更新することで通信がブロックされることが確認できます。

   .. image:: ./media/dcs-ratelimit-browser.jpg
       :width: 400

エラーページが画面に表示されます。
またブラウザの開発者ツールを開き、リクエストの詳細を確認すると、Curlコマンドと同様に Status Code 429が応答されていることが確認できます


4. RateLimit の解除
====

その他の機能を確認するため設定を解除する手順です。

`こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module08/module08.html#ratelimit>`__ の手順を参考に、HTTP Load Balancerに割り当てたRate Limitの設定を解除してください

   .. image:: ./media/dcs-ratelimit-disable.jpg
       :width: 400

5. Terraform を用いた HTTP Load Balancer + RateLimit の作成
====

ここで紹介したHTTP load Balancer + RateLimit を Terraform を使ってデプロイすることが可能です。

Terraform の利用で必要となる事前作業については `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module03/module03.html>`__ の手順を参考してください

パラメータの指定
----

実行に必要なファイル、また実行環境に合わせたパラメータを指定してください

.. code-block:: bash
  :linenos:
  :caption: terraform 実行前作業

  $ git clone https://github.com/BeF5/f5j-dc-waap-automation
  $ cd f5j-dc-waap-automation/terraform/rate-limit

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
----

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


6. API を用いた HTTP Load Balancer + RateLimit の作成
====

ここで紹介したHTTP load Balancer + RateLimit を API を使ってデプロイすることが可能です。

API の利用で必要となる事前作業については `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module03/module03.html>`__ の手順を参考してください

以下マニュアルを参考に、パラメータを指定して実行してください。

- HTTP Load Balancer

  - `API for http_loadbalancer <https://docs.cloud.f5.com/docs/api/views-http-loadbalancer>`__
  - `Example of creating http_loadbalancer <https://docs.cloud.f5.com/docs/reference/api-ref/ves-io-schema-views-http_loadbalancer-api-create>`__

    - ページ中段 ``Request using curl`` をご覧ください

送付するJSON データの書式は実際に作成したコンフィグのJSONデータからも確認をいただけます。合わせてご確認ください

パラメータの指定
----

GitHubよりファイルを取得します。 ``rate-limit-httplb.json`` をAPIの値として指定します。
``**<変数名>**`` が環境に合わせて変更するパラメータとなります。適切な内容に変更してください。

``Originl Pool Object`` は HTTP Load Balancer の Originl Pool 作成手順に従って作成ください

APIの利用
----

以下のサンプルを参考にAPIを実行してください。
証明書のファイル名、パスワード情報は適切な内容を指定してください。

- ファイル取得

.. code-block:: bash
  :linenos:
  :caption: APIによるオブジェクトの作成

  $ git clone https://github.com/BeF5/f5j-dc-waap-automation
  $ cd f5j-dc-waap-automation/api/rate-limit

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
       -d @rate-limit-httplb.json


- オブジェクトの削除

.. code-block:: bash
  :linenos:
  :caption: APIによるオブジェクトの削除

  # HTTP LB の削除
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/http_loadbalancers/**httplb_name** \
       --cert **/path/to/api_credential.p12-file**:**password** \
       --cert-type P12 \
       -X DELETE

  # Origin Pool の削除
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/origin_pools/**op_name** \
       --cert **/path/to/api_credential.p12-file**:**password** \
       --cert-type P12 \
       -X DELETE
