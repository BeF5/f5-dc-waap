F5 DCS WAAP 構成と事前作業
####

1. F5 DCS WAAP を設定する方法
====

F5 DCS WAAP は以下の方法で設定することが可能です

- コンソール画面からGUIで操作し、オブジェクトの作成
- Terraform Provider を利用したオブジェクトの作成
- API を利用したオブジェクトの作成

用途に合わせてご利用ください

2. F5 DCS WAAPの構成
====

F5 DCS WAAPの構成について紹介します。

   .. image:: ./media/dcs-waap-lab-diagram.JPG
       :width: 400

こちらに示している各種機能をF5 DCSのコンソール画面から設定します

| F5 DCSには ``Tenant`` と ``Namespace`` があり、その配下で各種設定オブジェクトを管理します。
| 契約者毎に ``Tenant`` が割り当てられます。あるTenantに所属するユーザは、そのTenanat内に ``Namespace`` を作成することが可能です
| また、ユーザが定義する Namespace の他に、いくつかの Namespace が存在します
| 詳細は、 `Core Concepts <https://docs.cloud.f5.com/docs/ves-concepts/core-concepts>`__ を参照してください。

   .. image:: ./media/dcs-waap-tenant-ns.JPG
       :width: 600

その他WAAPの設定に関連するオブジェクトを示します。
こちらの例ではユーザが定義した2つの Namespace にそれぞれHTTP Load Balancerを構成しています。
HTTP Load Balancerはその提供機能に応じた設定パラメータを持ちます。各機能は、HTTP Load Balancerの設定項目としてパラメータを指定します。
一部の設定については、Namespace 内で別の 設定オブジェクト として定義され、それらを参照する構成をとります。
HTTP Load Balancerの外部で定義された 設定オブジェクト は同一Namespace内の別のHTTP Load Balancerから参照可能です。

また、一部の設定オブジェクトについては、Shared Object として作成することが可能です。このオブジェクトは、複数のName Spaceから参照することができます。

   .. image:: ./media/dcs-waap-objects.JPG
       :width: 600

3. Namespaceの作成
====

本ラボで利用する ``Namespace`` を別に作成する場合、新規に作成頂くことが可能です。
すでに利用できる ``Namespace`` があり、新規に作成が不要である場合、こちらの手順をスキップしてください

F5 DCS のコンソールを開き、 ``Administration`` を開きます

   .. image:: ./media/dcs-console-administration.JPG
       :width: 400

Personal Management の ``My Namespaces`` を開き、上部に表示される ``Add namespaces`` をクリックしてください

   .. image:: ./media/dcs-waap-add-namespace.JPG
       :width: 400

表示される項目を入力し、 ``Save changes`` をクリックしてください

   .. image:: ./media/dcs-waap-add-namespace2.JPG
       :width: 400

4. Tenant ID等の確認
====

ご利用されるアカウントのテナントID等の情報は以下の手順でご確認いただけます。
それぞれの情報はTerraform/APIなどで利用いたします。利用の際にはこちらの項目をご確認ください。

F5 DCS のコンソールを開き、 ``Administration`` を開きます

   .. image:: ./media/dcs-console-administration.JPG
       :width: 400

画面左側 ``Tenant Settings`` の ``Tenant Overview`` を開き、画面に表示される内容を確認してください。

   .. image:: ./media/dcs-administration-tenant-information.jpg
       :width: 400
