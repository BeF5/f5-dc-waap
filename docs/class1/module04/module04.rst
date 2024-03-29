XC HTTP LB の設定
####

※本資料の画面表示や名称は資料作成時点の画面表示を利用しております。アップデート等より表示が若干異なる場合がございます。


基本となるHTTP Load Balancerを設定します。このサンプルではHTTPSで待ち受けるLBを設定します。

マニュアルは以下のページを参照してください。

- `HTTP Load Balancer <https://docs.cloud.f5.com/docs/how-to/app-networking/http-load-balancer>`__

1. HTTP Load Balancerの設定
====

1. HTTPS の設定
----

メニューより ``Load Balancers`` を選択してください

   .. image:: ./media/dcs-console-lb.JPG
       :width: 400

.. NOTE::
    すでにメニューを開いている場合、画面左側の ``Select service`` からメニューを開くことも可能です
    
   .. image:: ./media/dcs-menu.jpg
       :width: 400


| 左メニュー ``Namespace`` より、これから作成するオブジェクトを配置する Namespace を選択してください。
| サンプルに示している通り、テキストボックスに文字列を入力することにより検索が可能です。

   .. image:: ./media/dcs-lb-ns.JPG
       :width: 400

新規にHTTP Load Balanceを作成します。左メニュー ``HTTP Load Blancers`` をクリックし、 ``Add HTTP Load Balancer`` をクリックします

   .. image:: ./media/dcs-lb-new.JPG
       :width: 400

以下の通りパラメータを入力します。
FQDNについては後ほど適切にアプリケーションにアクセス出来るよう設定します。

-  入力パラメータ

   =========================== =============================
   Name                        demo-echo-lb
   --------------------------- -----------------------------
   List of Domain              echoapp.f5demo.net
   --------------------------- -----------------------------
   Select Type of Load Blancer HTTPS with Custom Certificate
   =========================== =============================

    .. image:: ./media/dcs-lb-conf1.jpg
        :width: 400


.. NOTE::
   Select Type of Load Blancer の項目では以下のようなパラメータが選択可能です

   ================================ ====================================================
   項目名                            用途
   ================================ ====================================================
   HTTP                             HTTP Load balancer
   -------------------------------- ----------------------------------------------------
   HTTPS with Automatic Certificate | 証明書の自動更新を提供します。この設定を選択する場合、
                                    | XCにドメインのDelegateをしている必要があります。
   -------------------------------- ----------------------------------------------------
   HTTPS with Custom Certificate    別途ご用意いただいた証明書をご利用いただけます。
   ================================ ====================================================

HTTPSに利用する ``証明書`` と ``鍵`` をアップロードします。 ``Select Type of Load Blancer`` の ``HTTP Loadbalancer TLS Parameters`` 欄の ``Configure`` をクリックしてください

   .. image:: ./media/dcs-lb-tls.jpg
       :width: 400

TLS設定の画面に遷移します。 ``Add Item`` をクリックします

   .. image:: ./media/dcs-lb-tls2.jpg
       :width: 400

``Certificate`` に証明書の内容を貼り付けます。
``Private Key`` 欄の ``Configure`` をクリックし、鍵を登録します。

   .. image:: ./media/dcs-lb-tls3.jpg
       :width: 400

``Secret Info`` で ``Clear Secret`` を選択し、下に表示されるテキストボックスに鍵の情報を貼り付け、 ``Apply`` をクリックします

   .. image:: ./media/dcs-lb-tls4.jpg
       :width: 400

画面下部の ``Add Item`` をクリックします

   .. image:: ./media/dcs-lb-tls5.jpg
       :width: 400

画面下部の ``Apply`` をクリックします

   .. image:: ./media/dcs-lb-tls6.jpg
       :width: 400

2. 分散先の設定
----

次に、Origin Poolsを指定します。画面を下にスクロールし ``Origin Pools`` のメニューを表示してください。
画面中央の ``Add Item`` をクリックします

   .. image:: ./media/dcs-origin-pool.JPG
       :width: 400

Origin Poolの選択画面が表示されます。これから新規にOrigin Poolを作成しますので、Origin Pool選択欄から ``Create new origin pool`` をクリックします

   .. image:: ./media/dcs-origin-pool2.JPG
       :width: 400

| ``Name`` 欄に ``demo-origin-pool`` と入力します。
| 新たに分散先のサーバを追加します。 ``Origin Serves`` に表示される ``Add Item`` をクリックします

   .. image:: ./media/dcs-origin-pool3.jpg
       :width: 400

以下の内容で転送先サーバを追加します。サーバを追加し、 ``Add Item`` をクリックしてください。
この操作を追加対象のサーバ台数分繰り返してください。

- Select Type of Origin Server

    ================================ ========================================
    Public DNS Name of Origin Server 対象の分散先サーバをDNS(FQDN)で指定する場合
    Public IP of Origin Server       対象の分散先サーバをIPアドレスで指定する場合
    ================================ ========================================

   .. image:: ./media/dcs-origin-pool4.jpg
       :width: 400

分散先サーバが待ち受けるポートを指定します。このサンプルアプリケーションでは ``80`` を指定します
内容を確認し、 ``Continue`` をクリックします

   .. image:: ./media/dcs-origin-pool5.jpg
       :width: 400

   .. image:: ./media/dcs-origin-pool6.jpg
       :width: 400

.. NOTE::
   分散先サーバがHTTPSを利用する場合、分散先サーバの待ち受けるポートを ``443`` と指定し、TLS Configurationで ``TLS`` を選択し、適切なオプションを指定してください


``Add Item`` をクリックし、Origin Pool の追加を完了します

   .. image:: ./media/dcs-origin-pool7.jpg
       :width: 400

画面最下部まで移動し、 ``Save and Exit`` をクリックします

   .. image:: ./media/dcs-lb-save.jpg
       :width: 400

設定した内容が画面に表示されます

   .. image:: ./media/dcs-lb-done.jpg
       :width: 400

2. 動作確認
====

1. クライアントのhostsファイルを変更
----

設定したHTTPSサイトに接続するため、クライアントのhostsファイルを変更します

   .. image:: ./media/dcs-origin-cname-copy.jpg
       :width: 400

.. NOTE::
    hostsファイルを利用せず、DNSのレコードを変更する場合、CNAMEの内容をDNSサーバに登録してください

CNAME欄に指定されたFQDNのアドレスをDNSサーバで解決し、IPアドレスを取得します

.. code-block:: bash
  :linenos:
  :caption: dig コマンドによるIPアドレス解決の結果
  :emphasize-lines: 2

  # dig ves-io-101f0be3-de90-4c78-8a1e-a101ce0336bd.ac.vh.ves.io +short
  72.19.3.189

表示されたIPアドレスを、アクセスするFQDN ``echoapp.f5demo.net`` のIPアドレスとしてhostsファイルに登録してください

.. code-block:: bash
  :linenos:
  :caption: hosts ファイル登録例

  72.19.3.189 echoapp.f5demo.net


2. クライアントから動作確認
----

ブラウザから ``https://echoapp.f5demo.net`` へアクセスします。後ほど、コンソールから接続結果を確認するため複数回アクセスしてください

   .. image:: ./media/dcs-sample-access.jpg
       :width: 400

3. 接続結果の確認
----

接続結果を確認します。

画面左側、Virtual Hostsの ``HTTP LoadBalancers`` をクリックし、
``demo-echo-lb`` の下にマウスを移動し、表示される ``Performance Monitoring`` のメニューをクリックしてください

   .. image:: ./media/dcs-lb-performance.jpg
       :width: 400

``Dashboard`` が表示されます。その他にも様々な結果を確認することができますので操作してみてください。
また、画面右上に対象とする期間の指定や、最新情報へ更新することが可能ですのでご希望の内容を確認してください

   .. image:: ./media/dcs-lb-performance2.jpg
       :width: 400

次に画面左側、Meshの ``Service Mesh`` をクリックし、表示された項目の ``More`` をクリックします

   .. image:: ./media/dcs-lb-mesh.jpg
       :width: 400

.. NOTE::
    対象のHTTP Load BalancerにLabelの割当がない場合、Namespace 名で項目が表示されます。Labelの割当がある場合、Labelが項目の名称として表示されます
    指定した期間にNamespaceやLabelなど複数のオブジェクトに対して通信がある場合、それらが項目として表示されます。

こちらではService Graphなどアプリケーションの通信状態など詳細を把握することが可能です。
引き続き設定を紹介いたしますので、適宜各ダッシュボードの内容を確認し、XCから把握できる通信の情報をご覧ください

   .. image:: ./media/dcs-lb-mesh2.jpg
       :width: 400

3. Terraform を用いた HTTP Load Balancer の作成
====

ここで紹介したHTTP load Balancer を Terraform を使ってデプロイすることが可能です。

Terraform の利用で必要となる事前作業については `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module03/module03.html>`__ の手順を参考してください

パラメータの指定
----

実行に必要なファイル、また実行環境に合わせたパラメータを指定してください

.. code-block:: bash
  :linenos:
  :caption: terraform 実行前作業

  $ git clone https://github.com/BeF5/f5j-dc-waap-automation
  $ cd f5j-dc-waap-automation/terraform/http-load-balancer

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

4. API を用いた HTTP Load Balancer の作成
====

ここで紹介したHTTP load Balancer を API を使ってデプロイすることが可能です。
APIでオブジェクトを作成する場合、Origin PoolとHTTP Load Balancerを作成する必要があります。

API の利用で必要となる事前作業については `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module03/module03.html>`__ の手順を参考してください

以下マニュアルを参考に、パラメータを指定して実行してください。

- Origin Pool

  - `API for origin_pool <https://docs.cloud.f5.com/docs/api/views-origin-pool>`__
  - `Example of creating origin_pool <https://docs.cloud.f5.com/docs/reference/api-ref/ves-io-schema-views-origin_pool-api-create>`__

    - ページ中段 ``Request using curl`` をご覧ください

- HTTP Load Balancer

  - `API for http_loadbalancer <https://docs.cloud.f5.com/docs/api/views-http-loadbalancer>`__
  - `Example of creating http_loadbalancer <https://docs.cloud.f5.com/docs/reference/api-ref/ves-io-schema-views-http_loadbalancer-api-create>`__

    - ページ中段 ``Request using curl`` をご覧ください

送付するJSON データの書式は実際に作成したコンフィグのJSONデータからも確認をいただけます。合わせてご確認ください


パラメータの指定
----

GitHubよりファイルを取得します。 ``base-httplb.json`` と ``base-origin-pool.json`` をAPIの値として指定します。
``**<変数名>**`` が環境に合わせて変更するパラメータとなります。適切な内容に変更してください。

APIの利用
----

以下のサンプルを参考にAPIを実行してください。
証明書のファイル名、パスワード情報は適切な内容を指定してください。

- ファイル取得

.. code-block:: bash
  :linenos:
  :caption: APIによるオブジェクトの作成

  $ git clone https://github.com/BeF5/f5j-dc-waap-automation
  $ cd f5j-dc-waap-automation/api/http-load-balancer

- オブジェクトの作成

.. code-block:: bash
  :linenos:
  :caption: APIによるオブジェクトの作成

  # Originl Pool の作成
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/origin_pools \
       --cert **/path/to/api_credential.p12-file**:**password** \
       --cert-type P12 \
       -X POST \
       -d @base-origin-pool.json

  # HTTP LB の作成
  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/http_loadbalancers \
       --cert **/path/to/api_credential.p12-file**:**password** \
       --cert-type P12 \
       -X POST \
       -d @base-httplb.json


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
