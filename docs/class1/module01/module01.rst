F5 DCS 環境への接続
####


以下の手順に従って F5 DCS のコンソール画面へ接続します  

1. F5 DCSへログイン
====

| `F5 DCS Top <https://www.f5.com/ja_jp/cloud>`__ ページへアクセスします
| 画面右上の ``Log in`` から F5 DCSへログインしてください
| アカウントを持っていない方は、``Sign up`` よりアカウントを登録するか、弊社担当営業までご連絡ください
| アカウントの新規作成は、 `こちら <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module01/module01.html#tips1>`__ を参照してください

   .. image:: ./media/dcs-f5cloud-top.jpg
       :width: 400

ログインするアカウントに合わせてメニューを選択し、ログインを進めてください

=============================== ============================================
For Free or Individual accounts 個人アカウントでログインする場合
------------------------------- --------------------------------------------
I'm a member of a team account  組織のアカウントサーバと連携し、ログインする場合
=============================== ============================================

   .. image:: ./media/dcs-login.JPG
       :width: 400


正しくログインが完了すると、以下のようにコンソール画面が表示されます

   .. image:: ./media/dcs-console.JPG
       :width: 400


Tips1. アカウントの新規作成
====

| アカウントをお持ちでない場合、新規にアカウントを作成し、``F5 DCS`` にログイン頂くことが可能です
| Freeプランでの作成方法を示します。利用できる機能は限定的ですが、``F5 DCS`` のコンソール画面などご覧いただけます

| `F5 DCS Top <https://www.f5.com/ja_jp/cloud>`__ ページへアクセスします
| 画面右上の ``Log in`` をクリックします

   .. image:: ./media/dcs-f5cloud-top.jpg
       :width: 400

| 画面下部の ``Sign up`` をクリックします

   .. image:: ./media/dcs-login.JPG
       :width: 400


| 画面一番左の ``Free Plan`` 中段にある ``Select Plan`` をクリックし、画面下部のチェックボックスにチェックし、 ``Next`` をクリックしてください
| こちらのチェックボックスで同意頂く内容について参考情報として以下に示します

- `END User Service Agreement <https://www.f5.com/pdf/customer-support/eusa.pdf>`__
- `Privacy Policy <https://www.f5.com/company/policies/privacy-notice>`__

   .. image:: ./media/dcs-new-selectplan.JPG
       :width: 400

アカウントのログインIDとして利用するメールアドレスを入力し、 ``Next`` をクリックしてください

   .. image:: ./media/dcs-new-mail.JPG
       :width: 400

画面に表示された内容に従って、作成するアカウントの情報を入力してください。入力が完了したら、 ``Create account`` をクリックしてください

   .. image:: ./media/dcs-new-accountinfo.JPG
       :width: 400

操作が完了すると以下のような画面が表示されます。入力したメールアドレスにメールアドレス確認が通知されますのでメールボックスを確認してください。

   .. image:: ./media/dcs-new-sendmail.JPG
       :width: 400

以下のようなメールを受信しますので、メール本文の ``Update Password`` をクリックしてください。
メールは一定時間でExpireしますのでご注意ください。

   .. image:: ./media/dcs-new-mail-updatepassword.JPG
       :width: 400

リンクをクリックするとパスワード設定画面が表示されます。画面に表示されるパスワードポリシーに従ってパスワードを設定してください。

   .. image:: ./media/dcs-new-updatepassword.JPG
       :width: 400

正しくパスワードの設定画完了するとログイン画面へのボタンが表示されますので、 ``Log in`` をクリックして F5 DCS にログインしてください。

   .. image:: ./media/dcs-new-updatepassword2.JPG
       :width: 400

``For Free or Individual accounts`` をクリックし、登録したアカウントとパスワードでログインしてください。

   .. image:: ./media/dcs-login.JPG
       :width: 400

| チュートリアルが表示されますので適宜情報を選択し、最後に ``Get Started`` をクリックしてください
| (こちらで選択した内容はいつでも `こちらアカウント設定 <https://f5j-dc-waap.readthedocs.io/ja/latest/class1/module01/module01.html#tips3-work-domains-and-skill-levels>`__ から変更可能です)

-    .. image:: ./media/dcs-new-tutorial1.JPG
       :width: 400

-    .. image:: ./media/dcs-new-tutorial2.JPG
       :width: 400

コンソールが表示されます。希望する操作を行ってください。

   .. image:: ./media/dcs-new-console.JPG
       :width: 400


Tips2. サポートチケットのオープン
====

| F5 DCS のコンソールでは右上の ``Support`` から様々な操作を頂くことが可能です。日々のご利用で必要となるメニューを適宜ご利用ください。
| またF5 DCSの操作に関する不明点や、想定外の操作となった場合にはサポートチケットを用いて問い合わせを頂くことが可能です

    .. image:: ./media/dcs-support-contact-support.JPG
       :width: 400

各項目を適切に入力し、ご要望の内容を適宜問い合わせてください。

    .. image:: ./media/dcs-support-contact-support2.JPG
       :width: 400

Tips3. チュートリアルで指定した Skill / Level の変更
====

画面右上にログインしているアカウントに関するメニューがあります。
``Account Settings`` をクリックするとアカウントの設定に関するメニューが表示されます。

    .. image:: ./media/dcs-new-console2.JPG
       :width: 400

``Work domains and skill levels`` から先程チュートリアルで指定した内容などこちらからご希望の内容となるように適宜変更ください。

    .. image:: ./media/dcs-new-console-skilllevels.JPG
       :width: 400

Tips4. 各プランの提供内容の違い
====

各プランの提供内容の違いは以下のドキュメントを参照してください。

- `プライシング｜F5 Distributed Cloud Services <https://www.f5.com/ja_jp/cloud/pricing>`__
