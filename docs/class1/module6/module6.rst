F5 DCS Bot 対策機能 (作成中)
####

F5 DCS で Bot 対策機能 を利用する方法や、各種設定について紹介します

1. F5 DCS Bot 対策機能 について
====

F5 DCS Bot 対策機能は、以下の特徴を備えております

- 従来のBot検出技術を回避する高度なBot （Captchaバイパスや、ブラウザ偽装ツール等を利用したBot）を、独自に開発した人工知能と機械学習モデルを用いて高精度に判定
- ログイン、サインアップ画面などの重要なフローを保護し

F5 DCS WAAPはこれらの高度なセキュリティをアプリケーションの迅速な展開に合わせて自由にご利用いただける環境を実現します

2. Bot 対策機能 の設定
====

1. Bot Defence Config の設定
----

作成済みのHTTP Load Balancerに作成で Bot Defence Config を設定します
HTTP Load Balancer の設定手順は `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module3/module3.html>`__ を参照ください


画面左側 Manage欄の ``Load Balancers`` 、 ``HTTP Load Balancers`` を開き、対象のLoad Balancerを表示し画面右側に遷移します。

   .. image:: ../module5/media/dcs-edit-lb.jpg
       :width: 400

すでに作成済みのオブジェクトを変更する場合、対象のオブジェクト一番右側 ``‥`` から、 ``Manage Configuration`` をクリックします

   .. image:: ../module5/media/dcs-edit-lb2.jpg
       :width: 400

設定の結果が一覧で表示されます。画面右上 ``Edit Configuration`` から設定の変更します。 
Security COnfiguration 欄の ``Bot Defense Config`` から設定します

   .. image:: ./media/dcs-edit-lb-bot.jpg
       :width: 400

   .. image:: ./media/dcs-edit-lb-bot2.jpg
       :width: 400

   .. image:: ./media/dcs-edit-lb-bot3.jpg
       :width: 400

   .. image:: ./media/dcs-edit-lb-bot4.jpg
       :width: 400

   .. image:: ./media/dcs-edit-lb-bot5.jpg
       :width: 400

   .. image:: ./media/dcs-edit-lb-bot6.jpg
       :width: 400

   .. image:: ./media/dcs-edit-lb-bot7.jpg
       :width: 400

2. 動作確認
====

1. 正常動作
----

2. Curlコマンドによるアクセス
----

3. ブラウザ自動操作ツールによるアクセス

