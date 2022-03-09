F5 DCS WAAP 構成と事前作業
####

1. F5 DCS WAAPの構成
====

F5 DCS WAAPの構成について紹介します。

   .. image:: ./media/dcs-waap-lab-diagram.jpg
       :width: 400

こちらに示している各種機能をF5 DCSのコンソール画面から設定します

F5 DCSには ``Tenant`` と ``Namespace`` があります
契約者毎に ``Tenant`` が割り当てられます。あるTenantに所属するユーザは、そのTenanat内に ``Namespace`` を作成することが可能です

   .. image:: ./media/dcs-waap-tenant-ns.jpg
       :width: 400

その他設定に関連するオブジェクトを参考に示します。こちらの例では2つのHTTP Load Balancerを構成しています。通信要件に応じて各種設定オブジェクトを用いて細かな設定が可能です。

   .. image:: ./media/dcs-waap-objects.jpg
       :width: 400

2. Namespaceの作成
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