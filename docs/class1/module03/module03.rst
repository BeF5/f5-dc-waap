XC WAAP Terraform/API の事前準備
####

※本資料の画面表示や名称は資料作成時点の画面表示を利用しております。アップデート等より表示が若干異なる場合がございます。

XC WAAP を操作するために必要な Terraform/API の事前準備
====

この章では、XC WAAP に関連するTerraform/APIの情報を紹介します

Terraform/API で利用する証明書の取得
----

Terraformを実行するホストでAPIに接続するための証明書が必要となります。証明書の作成方法を示します。
マニュアルは以下のページを参照してください。

- `Credentials <https://docs.cloud.f5.com/docs/how-to/user-mgmt/credentials>`__

XC のコンソールを開き、 ``Administration`` を開きます

   .. image:: ./media/dcs-console-administration.jpg
       :width: 400

Personal Management の ``Credentials`` を開き、上部に表示される ``Create Credentials`` をクリックします

   .. image:: ./media/dcs-create-credentials.jpg
       :width: 400

画面左側に表示される項目に各種情報を入力してください。 ``Credential Type`` は ``API Certificate`` を指定ください。パスワードは、Terraform を利用するホストの環境変数 ``VES_P12_PASSWORD`` に指定しますのでメモしてください。
他のパラメータは環境に合わせて自由に指定してください。

   .. image:: ./media/dcs-create-credentials2.jpg
       :width: 400

入力後、画面最下部の ``Download`` をクリックします。ポップアップでファイルのダウンロードを求められますので適当な場所に APIに用いる証明書を保存してください

XC WAAP で Terraform を利用する方法
====

Terraform ドキュメント
----

XC WAAP の Terraform は以下ドキュメントで紹介されています

- `XC Terraform <https://registry.terraform.io/namespaces/volterraedge>`__ 

画面右上の ``Documentation`` を開いてください

   .. image:: ./media/terraform-volterra.jpg
       :width: 400

利用の開始は記事中の内容に従って操作してください。
また、画面左側から Provider が提供する各種機能を参照できます

   .. image:: ./media/terraform-volterra2.jpg
       :width: 400

Terraform 証明書の利用
----

先述の手順で取得した証明書を、Terraformを実行するホストに保存し、Path情報をメモします。

こちらの証明書を利用する際、Terraformは環境変数の ``VES_P12_PASSWORD`` の値が、作成した証明書の値と一致する必要があります。実行する環境に合わせて環境変数を設定してください。以下はUbuntuの環境でbashの環境変数として指定する例です

.. code-block:: bash
  :linenos:
  :caption: 環境変数の指定

  $ export VES_P12_PASSWORD=**password-string**

F5 XC WAAP Terraform Provider
----

Provider を利用する際、Terraformで以下ように記述します

.. code-block:: bash
  :linenos:
  :caption: XC Provider を利用する方法

  provider "volterra" {
    api_p12_file     = "/path/to/api_credential.p12"
    url              = "https://<tenant_name>.console.ves.volterra.io/api"
  }

パラメータとして以下を指定します。

============= ==== ===============================================================================================================
api_p12_file  `-`  APIの認証情報として用いる、P12のファイルのPath情報
url           必須 XC の API Endopoint を示すURL。 ``tenant_name`` は
                   `Tenant Information <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module02/module02.html#tenant-id>`__
                   のCompany Nameに記載の文字列
============= ==== ===============================================================================================================

.. NOTE::
  Free Plan等、個人アカウントでAPIを利用する場合、URLは ``https://console.ves.volterra.io/api`` を指定してください。

その他詳細についてはマニュアルの内容を参照してください。

必要なパッケージの確認
----

XC Terraform の本書作成時点の対応バージョンは以下となります。

- Terraform >= 0.13.x

情報は以下を参照してください。
Provider は Terraform 実行時、自動的に取得しますのでこちらのページのBuildは不要です

- `Git terraform-provider-volterra <https://github.com/volterraedge/terraform-provider-volterra>`__

Terraformのインストール手順は以下を参照してください。

- `Download Terraform <https://www.terraform.io/downloads>`__

Terraform の動作確認
----

正しく動作することを確認します。
必要となるファイルを取得してください。

.. code-block:: bash
  :linenos:
  :caption: terraform initの実行結果
  :emphasize-lines: 5-7

  $ git clone https://github.com/BeF5/f5j-dc-waap-automation
  $ cd f5j-dc-waap-automation/terraform/connection-test

以下、 ``test.tf`` の内容を環境に合わせて修正してください。

.. code-block:: bash
  :linenos:
  :caption: test.tf の修正
  :emphasize-lines: 13,14,20

  $ vi test.tf
  
  terraform {
    required_providers {
      volterra = {
        source  = "volterraedge/volterra"
        version = "0.11.6"
      }
    }
  }
  
  provider "volterra" {
    api_p12_file = "**/path/to/api_credential.p12-file**"
    url          = "https://**tenant_name**.console.ves.volterra.io/api"
  }
  
  // example: create healthcheck object
  resource "volterra_healthcheck" "eample-dummy-hc" {
    name                = "dummy-health-check-t"
    namespace           = "**your-namespace**"
    timeout             = 3
    interval            = 15
    unhealthy_threshold = 1
    healthy_threshold   = 3
    http_health_check {
      use_origin_server_name = true
      path                   = "/"
      use_http2              = false
    }
  }



``terraform init`` を実行します。初回実行時、6-8行目に示す通り、Providerが取得されます

.. code-block:: bash
  :linenos:
  :caption: terraform initの実行結果
  :emphasize-lines: 6-8

  $ terraform init
  
  Initializing the backend...
  
  Initializing provider plugins...
  - Finding volterraedge/volterra versions matching "0.11.6"...
  - Installing volterraedge/volterra v0.11.6...
  - Installed volterraedge/volterra v0.11.6 (signed by a HashiCorp partner, key ID D9A99FF2F2E29E35)
  
  Partner and community providers are signed by their developers.
  If you'd like to know more about provider signing, you can read about it here:
  https://www.terraform.io/docs/cli/plugins/signing.html
  
  Terraform has created a lock file .terraform.lock.hcl to record the provider
  selections it made above. Include this file in your version control repository
  so that Terraform can guarantee to make the same selections by default when
  you run "terraform init" in the future.
  
  Terraform has been successfully initialized!
  
  You may now begin working with Terraform. Try running "terraform plan" to see
  any changes that are required for your infrastructure. All Terraform commands
  should now work.
  
  If you ever set or change modules or backend configuration for Terraform,
  rerun this command to reinitialize your working directory. If you forget, other
  commands will detect it and remind you to do so if necessary.


``terraform plan`` を実行します

.. code-block:: bash
  :linenos:
  :caption: terraform planの実行結果

  $ terraform plan
  
  Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
  symbols:
    + create
  
  Terraform will perform the following actions:
  
    # volterra_healthcheck.eample-dummy-hc will be created
    + resource "volterra_healthcheck" "eample-dummy-hc" {
        + healthy_threshold   = 3
        + id                  = (known after apply)
        + interval            = 15
        + name                = "dummy-health-check-t"
        + namespace           = "**your-namespace**"
        + timeout             = 3
        + unhealthy_threshold = 1
  
        + http_health_check {
            + path                      = "/"
            + request_headers_to_remove = []
            + use_http2                 = false
            + use_origin_server_name    = true
          }
      }
  
  Plan: 1 to add, 0 to change, 0 to destroy.
  
  ─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
  
  Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform
  apply" now.

``terraform apply`` を実行し、設定を反映します。

.. code-block:: bash
  :linenos:
  :caption: terraform planの実行結果
  :emphasize-lines: 33

  $ terraform apply
  
  Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
  symbols:
    + create
  
  Terraform will perform the following actions:
  
    # volterra_healthcheck.eample-dummy-hc will be created
    + resource "volterra_healthcheck" "eample-dummy-hc" {
        + healthy_threshold   = 3
        + id                  = (known after apply)
        + interval            = 15
        + name                = "dummy-health-check-t"
        + namespace           = "**your-namespace**"
        + timeout             = 3
        + unhealthy_threshold = 1
  
        + http_health_check {
            + path                      = "/"
            + request_headers_to_remove = []
            + use_http2                 = false
            + use_origin_server_name    = true
          }
      }
  
  Plan: 1 to add, 0 to change, 0 to destroy.
  
  Do you want to perform these actions?
    Terraform will perform the actions described above.
    Only 'yes' will be accepted to approve.
  
    Enter a value: yes   <<< yes と入力する
  
  volterra_healthcheck.eample-dummy-hc: Creating...
  volterra_healthcheck.eample-dummy-hc: Creation complete after 1s [id=******]
  
  Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Applyが完了しました。コンソールを開き、正しくオブジェクトが作成されたことを確認します

   .. image:: ./media/dcs-terraform-apply-dummy.jpg
       :width: 400

``terraform destroy`` を実行し、設定を削除します

.. code-block:: bash
  :linenos:
  :caption: terraform destroyの実行結果
  :emphasize-lines: 38

  $ terraform destroy
  volterra_healthcheck.eample-dummy-hc: Refreshing state... [id=******]
  
  Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
  symbols:
    - destroy
  
  Terraform will perform the following actions:
  
    # volterra_healthcheck.eample-dummy-hc will be destroyed
    - resource "volterra_healthcheck" "eample-dummy-hc" {
        - annotations         = {} -> null
        - disable             = false -> null
        - healthy_threshold   = 3 -> null
        - id                  = "******" -> null
        - interval            = 15 -> null
        - labels              = {} -> null
        - name                = "dummy-health-check-t" -> null
        - namespace           = "**your-namespace**" -> null
        - timeout             = 3 -> null
        - unhealthy_threshold = 1 -> null
  
        - http_health_check {
            - headers                   = {} -> null
            - path                      = "/" -> null
            - request_headers_to_remove = [] -> null
            - use_http2                 = false -> null
            - use_origin_server_name    = true -> null
          }
      }
  
  Plan: 0 to add, 0 to change, 1 to destroy.
  
  Do you really want to destroy all resources?
    Terraform will destroy all your managed infrastructure, as shown above.
    There is no undo. Only 'yes' will be accepted to confirm.
  
    Enter a value: yes   <<< yes と入力する
  
  volterra_healthcheck.eample-dummy-hc: Destroying... [id=******]
  volterra_healthcheck.eample-dummy-hc: Destruction complete after 1s
  
  Destroy complete! Resources: 1 destroyed.
  ubuntu@ip-10-0-11-227:~/temp2$ cat test.tf
  terraform {
    required_providers {
      volterra = {
        source  = "volterraedge/volterra"
        version = "0.11.6"
      }
    }
  }
  
  provider "volterra" {
    api_p12_file = "**/path/to/api_credential.p12-file**"
    url          = "https://**tenant_name**.console.ves.volterra.io/api"
  }
  
  // example: create healthcheck object
  resource "volterra_healthcheck" "eample-dummy-hc" {
    name                = "dummy-health-check-t"
    namespace           = "**your-namespace**"
    timeout             = 3
    interval            = 15
    unhealthy_threshold = 1
    healthy_threshold   = 3
    http_health_check {
      use_origin_server_name = true
      path                   = "/"
      use_http2              = false
    }
  }


削除の結果を確認します。

   .. image:: ./media/dcs-terraform-destroy-dummy.jpg
       :width: 400


Terraformを使って正しく、追加、削除が出来ることが確認できました



XC WAAP で API を利用する方法
====

API ドキュメント
----

XC WAAP の API は以下ドキュメントで紹介されています

- `F5 Distributed Cloud Services API <https://docs.cloud.f5.com/docs/api>`__ 

APIのEndpointは以下の内容で指定いただくことが可能です。

.. code-block:: bash
  :caption: 環境変数の指定

  **METHOD**  https://**tenant_name**.console.ves.volterra.io/api/**service_prefix**/namespaces/**namespace**/

各要素は以下の通りです。

 ============== ==================================================================================================================
 METHOD         実行するHTTP Method。GET / PUT / POST / DELETE 等 用途に合わせて指定する
 tenant_name    APIの対象となるTenant名。
                `Tenant Information <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module02/module02.html#tenant-id>`__
                のCompany Nameに記載の文字列
 service_prefix | 対象となるサービスの名称。以下が主要なPrefix。詳細は各API Documentを参照のこと
                | ・ ``/api/config/``  - 設定オブジェクトに対するCRUD操作
                | ・ ``/api/data/``    - モニタリング等アナリティクス用途
                | ・ ``/api/ml/data/`` - Machine Learningに関する情報取得用途
 namespace      対象となるネームスペースの名称
 ============== ==================================================================================================================

.. NOTE::
  Free Plan等、個人アカウントでAPIを利用する場合、URLは ``https://console.ves.volterra.io/api/(利用に応じたPATH)`` を指定してください。

API 証明書の利用
----

先述の手順で取得した証明書を、APIで参照し利用します。
証明書の利用方法や、オプション、APIクライアントが対応する証明書の形式などは適宜マニュアルを参照してください。

Curlコマンドで実行する場合のサンプルを以下に示します

.. code-block:: bash
  :linenos:
  :caption: Curlコマンドを利用した Object の作成サンプル

  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/config/namespaces/**namespace**/http_loadbalancers \
         --cert **/path/to/api_credential.p12-file**:**password** \
         --cert-type P12 \
         -X POST \
         -d @**OBJECT-CONFIGURATION-JSON**.json

- ``1行目`` : URLで対象となるテナント、Namespaceを指定しています
- ``2行目`` : 利用するP12ファイルのPATHを指定しています。またコロン(:)に続いて、証明書作成時に指定したパスワードを指定します
- ``3行目`` : 証明書タイプをP12と指定しています
- ``4行目`` : 新規作成のため、POST Methodを指定しています
- ``5行目`` : 対象となるサービスで作成するオブジェクトの設定内容をJSONで送付します

要件に応じてMethodやパラメータを変更することで、設定の追加、変更、削除、ステータスの確認などを行うことが可能です

API の接続確認
----

Curlコマンドを使った接続確認方法を示します。
``System`` namespace の情報をAPIで取得し、正しく表示されればAPIの接続が正しいことを確認できます

.. code-block:: bash
  :linenos:
  :caption: Curlコマンドを利用した APIの接続確認

  $ curl -k https://**tenant_name**.console.ves.volterra.io/api/web/namespaces/system \
         --cert **/path/to/api_credential.p12-file**:**password** \
         --cert-type P12 \
         -X GET

  {
    "object": null,
    "create_form": null,
    "replace_form": null,
    "resource_version": "********",
    "metadata": {
      "name": "system",
      "namespace": "",
      "labels": {
      },
      "annotations": {
      },
      "description": "",
      "disable": false
    },
    "system_metadata": {
      "uid": "********",
      "creation_timestamp": "********",
      "deletion_timestamp": null,
      "modification_timestamp": "********",
      "initializers": null,
      "finalizers": [
      ],
      "tenant": "**your tenant name**",
      "creator_class": "******",
      "creator_id": "",
      "object_index": 0,
      "owner_view": null
    },
    "spec": {
  
    },
    "status": [
    ],
    "referring_objects": [
    ]
  }

Tips1. Terraform / API の利用・調査方法について
====

XC は作成済みのオブジェクトがどのような構成情報となるかJSON形式で確認することが可能です。

すでに作成済みのオブジェクトの情報を確認します。対象のオブジェクト一番右側 ``‥`` から、 ``Manage Configuration`` をクリックします

   .. image:: ./media/dcs-setting-edit.jpg
       :width: 400

表示された画面上部の ``JSON`` をクリックし、設定情報をJSON形式で確認します

   .. image:: ./media/dcs-setting-view-json.jpg
       :width: 400


Terraform での利用
----

Terraform Providerはドキュメントに詳細が記載されています。ドキュメントから利用方法を確認します

- `XC Terraform <https://registry.terraform.io/namespaces/volterraedge>`__ 

   .. image:: ./media/terraform-volterra2.jpg
       :width: 400

Terraformで作成されたいオブジェクトをGUIから実際に作成し、作成されたオブジェクトのJSON情報を参考にTerraform Providerの情報を確認すると効率的に調査を進めることが可能です。

また、同等の設定を持つオブジェクトのJSONを保存し、Terraform を通じて作成したObjectのJSONの値と比較することで同等の情報を持つか確認することができます。

API での利用
----

APIではこちらの画面で取得したJSONをPOSTで送付することで同じオブジェクトを作成することが可能です。

まずはJSONのデータでObjectを作成し、APIの振る舞いを確認し、その後不要な項目の削除やパラメータの確認を進めることができます。

