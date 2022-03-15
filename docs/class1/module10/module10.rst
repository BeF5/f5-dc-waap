F5 DCS DDoS
####

F5 DCS で DDoS について紹介します

こちらの記事はマニュアルページの解説です。実際のDDoS攻撃のエミュレートが難しいため、

マニュアルは以下のページの ``Monitor HTTP Load Balancer > Explore Security Monitoring > DDoS`` を参照してください
- `Monitor HTTP Load Balancer <https://docs.cloud.f5.com/docs/how-to/observe/monitor-http-load-balancer>`__


1. F5 DCS DDoS について
====

F5 DDoSは、以下の特徴を備えております

- APIスキーマとSwaggerファイルを生成し、APIエンドポイントの手動トラッキングを最小限に抑える
- 不審なリクエストをブロックし、データ漏洩を防止
- APIセキュリティポリシーの設定・導入にかかる時間を短縮

F5 DCS WAAPはこれらの高度なセキュリティをアプリケーションの迅速な展開に合わせて自由にご利用いただける環境を実現します

2. DDoS の設定・確認
====

2. Security Event の確認
----

以下の手順で Security Event を開いてください

   .. image:: ../module05/media/dcs-app-fw-log.jpg
       :width: 400

   .. image:: ../module05/media/dcs-app-fw-log2.jpg
       :width: 400

``DDoS`` のタブをクリックしてください。
最新の情報を取得するため、画面右上 ``Refresh`` をクリックしてください。

   .. image:: https://docs.cloud.f5.com/docs/static/bfb3a4141f0073e21a15728b366bbbe0/b779f/lb-ddos-map.webp
       :width: 400

画面下部に ``Timeline`` のタブが表示され、クリックすることでTimelineを表示することが可能です。
TimelineにはDDoSに関連する ``Request rate`` 、 ``Error rate`` を確認することが可能です

画面左上の ``DDoS Events`` をクリックすることにより、詳細を確認できます

   .. image:: https://docs.cloud.f5.com/docs/static/65bfacea7d563b278e7a1aa980661fc1/b779f/lb-ddos-events.webp
       :width: 400

表示されたEventの左側 ``>`` をクリックすることにより、JSON Formatで攻撃の詳細を確認することが可能です

画面右上の ``Analytics`` をクリックしてください。
攻撃元のIPアドレス、地域、AS番号、TLS Fingerprintsの統計を見ることができます。

各項目の ``∨`` をクリックすることにより、それぞれに含まれる要素を確認することが可能です。
要素を選択し、 ``Apply`` をクリックすることにより結果のフィルタが可能です

   .. image:: https://docs.cloud.f5.com/docs/static/9f0da6fdfbf8a59fc24154a0ccd1b7c7/b779f/lb-ddos-analytics.webp
       :width: 400

特定の要素を選択し、画面上部の ``Add Rule`` を選択することにより、DDoS 緩和に関するルールを HTTP Load Balancer に設定することが可能です。

   .. image:: https://docs.cloud.f5.com/docs/static/49917aeaac6b67bfaa5e15097f7e2a9f/b779f/lb-ddos-rules.webp
       :width: 400


.. NOTE::
    ``View Rules`` をクリックすることで、HTTP Load Balancerに設定されている DDoS ルールを確認することが可能です