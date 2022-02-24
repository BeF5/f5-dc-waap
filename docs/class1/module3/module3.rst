F5 DCS WAAP の設定
####


1. HTTP Load Balancerの設定
====

基本となるHTTP Load Balancerを設定します。このサンプルではHTTPSで待ち受けるLBを設定します

メニューより Load Balancers を選択してください

   .. image:: ./media/dcs-console-lb.JPG
       :width: 400

.. NOTE::

    すでにメニューを開いている場合、画面左側の ``Select service`` からメニューを開くことも可能です

    .. image:: ./media/dcs-menu.JPG
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

   ================================ =================================================================================================
   HTTP                             HTTP Load balancer
   -------------------------------- -------------------------------------------------------------------------------------------------
   HTTPS with Automatic Certificate | 証明書の自動更新を提供します。この設定を選択する場合、
                                    | F5 DCSにドメインのDelegateをしている必要があります。
   -------------------------------- -------------------------------------------------------------------------------------------------
   HTTPS with Custom Certificate    別途ご用意いただいた証明書をご利用いただけます。
   ================================ =================================================================================================

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


   .. image:: ./media/dcs-origin-pool.JPG
       :width: 400

   .. image:: ./media/dcs-origin-pool2.JPG
       :width: 400

   .. image:: ./media/dcs-origin-pool3.JPG
       :width: 400

   .. image:: ./media/dcs-origin-pool4.JPG
       :width: 400

   .. image:: ./media/dcs-origin-pool5.JPG
       :width: 400

   .. image:: ./media/dcs-origin-pool.JPG
       :width: 400

   .. image:: ./media/dcs-origin-pool.JPG
       :width: 400

   .. image:: ./media/dcs-origin-pool.JPG
       :width: 400

   .. image:: ./media/dcs-origin-pool.JPG
       :width: 400

   .. image:: ./media/dcs-origin-pool.JPG
       :width: 400

   .. image:: ./media/dcs-lb-save.JPG
       :width: 400

   .. image:: ./media/dcs-lb-done.JPG
       :width: 400

   .. image:: ./media/dcs-origin-cname-copy.jpg
       :width: 400

   .. image:: ./media/dcs-sample-access.jpg
       :width: 400

   .. image:: ./media/dcs-lb-performance.jpg
       :width: 400

   .. image:: ./media/dcs-lb-performance2.jpg
       :width: 400

