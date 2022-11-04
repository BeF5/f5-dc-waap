F5 DCS Bot 対策機能
####

F5 DCS で Bot 対策機能 を利用する方法や、各種設定について紹介します

マニュアルは以下のページを参照してください

- `Bot Defense <https://docs.cloud.f5.com/docs/how-to/advanced-security/bot-defense>`__


1. F5 DCS Bot 対策機能 について
====

F5 DCS Bot 対策機能は、以下の特徴を備えております

- 従来のBot検出技術を回避する高度なBot （Captchaバイパスや、ブラウザ偽装ツール等を利用したBot）を、独自に開発した人工知能と機械学習モデルを用いて高精度に判定
- ログイン、サインアップ画面などの重要なフローを保護します

F5 DCS WAAPはこれらの高度なセキュリティをアプリケーションの迅速な展開に合わせて自由にご利用いただける環境を実現します

2. Bot 対策機能 の設定
====

1. Bot Defense の設定方針
----

このBot対策機能では、保護対象のPATHへのリクエストに対する応答にJSを挿入し、クライアントの振る舞いが正規のユーザかどうかを評価いたします。
サンプルの手順では最低限の設定パラメータを紹介することを目的としています。

サンプルの設定では想定外の動作となり通信にエラーが発生する場合がありますので、実際のWebサイトを保護する際には以下のような方針を念頭に、PATHやMETHODを指定してください

- 保護する対象を整理し、適切なPATHを指定する。（/や、/*は非推奨）
- ブラウザよりGETメソッドで直接アクセスするパスについては、GET(Document)を指定する
- ブラウザよりPOSTメソッドでアクセスを行うパスについては、POSTを指定する

2. Bot Defence Config の設定
----

作成済みのHTTP Load Balancerに Bot Defence Config を設定します。
HTTP Load Balancer の設定手順は `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module04/module04.html>`__ を参照ください


画面左側 Manage欄の ``Load Balancers`` 、 ``HTTP Load Balancers`` を開き、対象のLoad Balancerを表示し画面右側に遷移します。

   .. image:: ../module06/media/dcs-edit-lb.jpg
       :width: 400

すでに作成済みのオブジェクトを変更する場合、対象のオブジェクト一番右側 ``‥`` から、 ``Manage Configuration`` をクリックします

   .. image:: ../module06/media/dcs-edit-lb2.jpg
       :width: 400

設定の結果が一覧で表示されます。画面右上 ``Edit Configuration`` から設定の変更します。 
Security Configuration 欄の ``Bot Defense Config`` から設定します

``Specify Bot Defense Configuration`` を選択し、 ``Bot Defense Regional Endopoint`` に ``Asia`` を選択し、その下に表示される ``Configure`` をクリックします。

   .. image:: ./media/dcs-edit-lb-bot.jpg
       :width: 400

Protected App Endopoints の App Endpoint Type 下に表示される ``Configure`` をクリックします。

   .. image:: ./media/dcs-edit-lb-bot2.jpg
       :width: 400

``Add Item`` をクリックします。

   .. image:: ./media/dcs-edit-lb-bot3.jpg
       :width: 400

以下のパラメータを入力します。

- 入力パラメータ

.. NOTE::
    | Method ``ANY`` は ``GET`` 、 ``POST`` 、 ``PUT`` を含みます。保護対象となるPrefixに XMLHttpRequest の宛先となる箇所が含まれる場合、Bot対策機能により想定と動作が異なる場合があります。
    | 対象のアプリケーションが意図した動作を示さない場合、`1. Bot Defense の設定方針 <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module07/module07.html#bot-defense>`__ の内容を参考に設定を見直してください。

============ =================
name         demo-bot-endpoint
HTTP Methods ANY
Prefix       /
============ =================

  .. image:: ./media/dcs-edit-lb-bot4.jpg
       :width: 400

各種設定を反映するため、画面右下の ``Apply`` をクリックし、最後にHTTP load Balancerの ``Save and Exit`` をクリックしてください

   .. image:: ./media/dcs-edit-lb-bot5.jpg
       :width: 400

   .. image:: ./media/dcs-edit-lb-bot6.jpg
       :width: 400

   .. image:: ./media/dcs-edit-lb-bot7.jpg
       :width: 400


3. Origin Server の変更
----

この例ではOrigin Serverとして `OWASP Juice Shop <https://owasp.org/www-project-juice-shop/>`__ を動作させます。OWASPが提供する脆弱なサーバとなりますので本テスト完了後、適切に停止させてください

Origin ServerでDockerを動作させ、以下コマンドでOWASP Juice Shopを ``80`` で待ち受けるよう設定してください

.. code-block:: bash
  :linenos:
  :caption: OWASP Juice Shop のデプロイ方法

   # OWASP Juice-shop を実行してください。初回はDocker Imageの取得のため起動に少し時間がかかります

   $ docker run -d --name dcs-juice-shop -p 80:3000 bkimminich/juice-shop 
   8b69c6f97763b7c08e4afde42942c046dcab400743d756fc36a833d7bb8fa507
   
   # 正しく起動していることを確認してください

   $ docker ps
   CONTAINER ID   IMAGE                   COMMAND                  CREATED         STATUS         PORTS                                   NAMES
   8b69c6f97763   bkimminich/juice-shop   "docker-entrypoint.s…"   3 seconds ago   Up 2 seconds   0.0.0.0:80->3000/tcp, :::80->3000/tcp   dcs-juice-shop

   # 利用が完了しましたら、対象のDocker Containerを停止してください
   $ docker stop $(docker ps -a -f name=dcs-juice-shop  -q)
   $ docker rm $(docker ps -a -f name=dcs-juice-shop  -q)


また、HTTP Load Balancer ではこの単一のOrigin Serverへ通信を転送するよう、Origin Pool を指定してください。

- Origin Pool の作成

   .. image:: ./media/dcs-lb-1-origin-pool.jpg
       :width: 400

- HTTP Load Balancer に Origin Pool の割当

   .. image:: ./media/dcs-lb-attach-1-origin-pool.jpg
       :width: 400


3. 動作確認
====


1. 正常動作
----

ブラウザで ``https://echoapp.f5demo.net`` にアクセスし、ページを閲覧してください
以下ログインアカウントでAdminとして動作できます。

    ================= =================
    username          admin@juice-sh.op
    ----------------- -----------------
    password          admin123
    ================= =================

   .. image:: ./media/dcs-js-login.jpg
       :width: 400


.. NOTE::
    | このサーバはセキュリティハックのトレーニング用のアプリケーションとなります。
    | 様々な操作が、セキュリティに関する操作に該当する場合があり、POP Upで得点を獲得した
    | 情報が表示されますが無視してください

    .. image:: ./media/dcs-js-popup.jpg
       :width: 400


正しくブラウザで操作が出来ることを確認してください。


2. ブラウザ自動操作ツールによるアクセス
----

ブラウザ自動操作ツールによるアクセスを確認します。
利用するツールはお客様環境に適したツールを自由に選択ください。

この例では、ブラウザ自動操作ツール( Selenium ) での動作を確認します。
今回のサンプルでは、 ``ログイン > 商品をポップアップで表示 > ログアウト`` を複数回繰り返す動作としております。


それでは通信の結果を確認します。

   .. image:: ../module06/media/dcs-app-fw-log.jpg
       :width: 400

   .. image:: ./media/dcs-app-bot-log.jpg
       :width: 400

   .. image:: ./media/dcs-app-bot-log2.jpg
       :width: 400


新たに ``Bot Defense`` 、 ``Bot Traffic Overview`` のタブが表示されます。

グラフの結果から、自動化ツールを使うことにより多くの通信が怪しいBotとして検知されていることがわかります

2. Curlコマンドによるアクセス
----

Top ページに対してCurlコマンドを実行します。その結果を確認します

.. code-block:: bash
  :linenos:
  :caption: OWASP Juice Shop のデプロイ方法

  $ while : ; do sleep 1 ; date ; curl -ks https://echoapp.f5demo.net/ | grep title ; done

それでは通信の結果を確認します。

   .. image:: ./media/dcs-app-bot-curl-log.jpg
       :width: 400

こちらの場合には、User Agentが ``curl/7.58.0`` と表示され、 ``Bot`` と検知されていることが確認できます

3. Bot をブロックする設定に変更
---

HTTP Load Balancer の設定を変更し、Botをブロックする設定とします。

   .. image:: ./media/dcs-app-bot-block.jpg
       :width: 400

   .. image:: ./media/dcs-app-bot-block2.jpg
       :width: 400


設定を反映した後、先程実行したCurlコマンドを停止させ、改めて以下コマンドでアクセスしてください

.. code-block:: bash
  :linenos:
  :caption: Curl コマンドを使った https://echoapp.f5demo.net への接続結果
  :emphasize-lines:  17

  $ curl -vks https://echoapp.f5demo.net/
  
  ** 省略 **
  
  > GET / HTTP/2
  > Host: echoapp.f5demo.net
  > User-Agent: curl/7.58.0
  > Accept: */*

  ** 省略 **

  < HTTP/2 200
  < content-type: text/html; charset=UTF-8

  ** 省略 **

  The requested URL was rejected. Please consult with your administrator.

先程設定変更をした内容の通り、Botに対して通信を拒否し、エラーメッセージが表示されていることを確認できます


4. Bot Defence Config の解除
====

その他の機能を確認するため設定を解除する手順です。

`こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module07/module07.html#bot-defence-config>`__ の手順を参考に、HTTP Load Balancerに割り当てたBot Defence Configを解除してください

   .. image:: ./media/dcs-bot-config-disable.jpg
       :width: 400

5. Terraform を用いた HTTP Load Balancer + Bot対策 の作成
====

ここで紹介したHTTP load Balancer + Bot対策 を Terraform を使ってデプロイすることが可能です。

Terraform の利用で必要となる事前作業については `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module03/module03.html>`__ の手順を参考してください

パラメータの指定
----

実行に必要なファイル、また実行環境に合わせたパラメータを指定してください

.. code-block:: bash
  :linenos:
  :caption: terraform 実行前作業

  $ git clone https://github.com/BeF5/f5j-dc-waap-automation
  $ cd f5j-dc-waap-automation/terraform/bot-protection

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


6. API を用いた HTTP Load Balancer + Bot対策 の作成
====

ここで紹介したHTTP load Balancer + Bot対策 を API を使ってデプロイすることが可能です。

API の利用で必要となる事前作業については `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module03/module03.html>`__ の手順を参考してください

以下マニュアルを参考に、パラメータを指定して実行してください。

- HTTP Load Balancer

  - `API for http_loadbalancer <https://docs.cloud.f5.com/docs/api/views-http-loadbalancer>`__
  - `Example of creating http_loadbalancer <https://docs.cloud.f5.com/docs/reference/api-ref/ves-io-schema-views-http_loadbalancer-api-create>`__

    - ページ中段 ``Request using curl`` をご覧ください

送付するJSON データの書式は実際に作成したコンフィグのJSONデータからも確認をいただけます。合わせてご確認ください

パラメータの指定
----

GitHubよりファイルを取得します。 ``bot-protection-httplb.json`` をAPIの値として指定します。
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
  $ cd f5j-dc-waap-automation/api/bot-protection

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
       -d @bot-protection-httplb.json


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

