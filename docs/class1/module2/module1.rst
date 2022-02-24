F5 DCS WAAP 
####

1. F5 DCS WAAPの構成
====

F5 DCS WAAPの構成について紹介します。

   .. image:: ./media/dcs-waap-lab-diagram.JPG
       :width: 400

こちらに示している各種機能をF5 DCSのコンソール画面から設定します

F5 DCSには ``Tenant`` と ``Namespace`` があります
契約者毎に ``Tenant`` が割り当てられます。あるTenantに所属するユーザは、そのTenanat内に ``Namespace`` を作成することが可能です

   .. image:: ./media/dcs-waap-tenant-ns.JPG
       :width: 400

その他設定に関連するオブジェクトを参考に示します。こちらの例では2つのHTTP Load Balancerを構成しています。通信要件に応じて各種設定オブジェクトを用いて細かな設定が可能です。

   .. image:: ./media/dcs-waap-objects.JPG
       :width: 400
