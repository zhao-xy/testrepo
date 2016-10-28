データベースstaging/productionサーバー構築手順
==============================================

サポートバージョン:

  対象               バージョン
  ------------------ ------------
  gemini webapp      -
  fenrir webapp      -
  antares database   1.0.0
  antares php api    1.2.1
  antares php impl   1.2.3
  libantares         1.1.0
  libsession         1.4.0
  fenrir-worker      -
  storage system     v4

更新日: 2015-11-09

<div class="legend">

凡例

``` {.command}
これは実際に実行するコマンドを示します。内容には {xxx} 形式で変数で示される場合があり、適切な値に置換します。
```

``` {.output}
これはコマンド実行の結果出力される内容です。
```

``` {.content}
編集前後のファイルの内容を示します。内容には {xxx} 形式で変数が、(...)で繰り返しが示される場合があり、適切な値に置換します。
```

</div>

<div class="section">

ドキュメントの概要：このドキュメントは
データベースのstagingもしくはproductionサーバーを構築する手順を示します。データベースプロトタイプサーバーから作成したAMIを元に、staging/production
サーバー固有の設定を追加していくかたちで行います。

[ドキュメントの更新方法](common/all.html#update-document){.server-common}

[内容に関する前提知識](common/all.html#expected-knowledge){.server-common}

変数の定義：このドキュメント内、または参照先の変数、{PROJECT} には db
、{ENV} には stagingの場合には stag、productionの場合には prod
をそれぞれ代入します。

</div>

<div class="section">

サーバーインスタンスを作成
--------------------------

AMI一覧に用意してあるdb
prototypeイメージの複数あるなかから適切なもの（同じ種類のものならその中から名前に最も新しい日付のついたもの）を選択しLaunchします。

[作成方法](common/launch-stag-m3m-instance.html#launch-stag-m3m-instance){.server-common}

  項目             値                                         備考
  ---------------- ------------------------------------------ --------------------------------------------------------------------------------------------------------------------------------------------------
  Instance Type    m3.medium                                  productionサーバーと同じタイプを選択します。
  Subnet           fenrir-ec2-1b-2                            privateサブネットを使用します。
  Volume           32                                         
  IAM role         ec2\_acweb\_db                             
  Tag Name         acweb-db-{storage\_version}-{ENV}-{番号}   {番号} は、このプロトタイプを再度構築する場合に重複しないように番号を1ずつ増加させます。{storage\_version} はストレージのバージョンです。例： v4
  Tag Department   acweb                                      
  Security Group   vpc-base-1, acweb-db-{ENV}                 staging/production用の Security Groupを追加します。

※上記以外の値はデフォルトの内容を選びます。

</div>

<div class="section">

アップデート
------------

この時点でのパッケージのアップデートを最後に以後原則しません（production
サーバーでパッケージアップデートをする場合にのみ、stagingサーバーもパッケージアップデートを行う）。

[アップデート方法](common/all.html#update){.server-common}

</div>

<div class="section">

プロンプト
----------

アプリケーションサーバーとは違い、データベースサーバーはユーザーを設定しません。

[変更方法](common/users-prompt.html#change-prompt){.server-common}

</div>
