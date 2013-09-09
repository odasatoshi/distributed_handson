AWSの設定
===============

ハンズオンを始める前に、Amazon web serviceにsign inしてください。以降、すべてAWSに接続されている状態であることを想定します。これから、ハンズオンを実施するにあたって必要な設定を行います。既にAWSを使いこなしている方は、適時設定を飛ばす、読み替える、などしていただいて構いません。

- クーポンによるチャージ：今回のハンズオンでは、アマゾン・ジャパン様のご厚意により、ハンズオンを実施するために十分なクーポンをいただいております。これをチャージして利用します。
- Key Pairの作成：今回のハンズオンでは、今日のハンズオン専用のAMI(Amazon Machine Image)を用います。この仮想マシンにログインするための鍵ペアを作成します。
- Security Groupsの作成、設定：今回のハンズオンは分散環境で動作します。同じ環境同士の通信は自由にできるように、かつ他の環境からは通信できないようにセキュリティグループを設定します。
- ハンズオン用のVMを起動：ハンズオン用のAMIから5台の仮想マシンを起動します。便宜上、``MANAGER``, ``C1``, ``C2``, ``S1``, ``S2`` と名づけます。
- VMへ接続：putty, TeraTerm, などSSHクライアントを用いて、環境に接続します。
 


クーポンによるチャージ
-------------------------
https://portal.aws.amazon.com/gp/aws/manageYourAccount

1. 上部、アカウント／コンソールから ``アカウント`` をクリック
2. 左のメニューから ``支払い方法`` をクリック
3. 右画面下部のAWS割引クーポンを引き換える、にある ``AWSクレジットを引き換え／表示`` をクリック
4. 下記の ``クレームコードを入力し、引き換えをクリックしてください。AWSアカウントに支払いを追加します。``  から指定されたコードを入力。
5. ただしくチャージされていれば、次のようになります。

.. image:: https://gist.github.com/odasatoshi/d95e62b070c1a679abd4/raw/e04c026e4a696c3345cda31c820436518bc9ff36/payments.png

EC2 ダッシュボード
------------------------
https://console.aws.amazon.com/ec2/v2/home

1. 上部、アカウント／コンソールから ``AWS Management Console``
2. EC2ダッシュボートをクリック

ここで、URLの ``region`` が、 ``ap-northeast`` となっていることを確認して下さい。
後ろに数字が付いているのは構いませんが、例えば ``us-west`` などになっていると、
以下のチュートリアルが実施できないことがあります。
右上の"あなたの名前"と ``Help`` の間にある地名が、 ``Tokyo`` になっていれば大丈夫です。

.. image:: https://gist.github.com/odasatoshi/d95e62b070c1a679abd4/raw/f03d68723ddcbc49c2d5f98fd1da860fac296d38/dashboard.png

Key Pairの作成
------------------------
https://console.aws.amazon.com/ec2/home?region=ap-northeast-1#s=KeyPairs

1. ダッシュボードの左側のメニュー  ``NETWORK & SECURITY``  から、 ``Key Pairs`` をクリック
2. Create Key Pairボタンをクリック
3. Key Pair Nameに ``jubatus_handson`` と入力して ``Create`` ボタンをクリック
4. ``jubatus_handson.pem`` というファイルがダウンロードされます。

このpemファイルはこれから作るサーバに接続するために必要な秘密鍵です。適時適切な所に配置してください。
Linux, Macなどをお使いの方は、ファイルPermissionの設定をする必要があります。具体的には

::

    chmod 600 jubatus_handson.pem

を実行しておかないと、sshクライアントが受け付けてくれません。
鍵は既にあるものを利用しても構いません。適時読み替えてください。

Security Groupsの作成、設定
-----------------------------
https://console.aws.amazon.com/ec2/home?region=ap-northeast-1#s=SecurityGroups

1. ダッシュボードの左側のメニュー ``NETWORK & SECURITY`` から、 ``Security Groups`` をクリック
2. Create Security Groupボタンをクリック
3. ``Name`` に ``jubatus_handson`` , ``Description`` に適当な説明を入力して ``Yes, Create`` ボタンをクリック
4. 作成したグループを選択し、下部の ``Inbound`` をクリック
5. Port Rangeに ``22`` を追加して、下のボタン、 ``Add Rule`` をクリック

.. image:: https://gist.github.com/odasatoshi/d95e62b070c1a679abd4/raw/b7cec527ba2d042c00a1f40bbc8c4871b147999a/secgroup.png

6. ``Create a new rule`` から、 ``All TCP`` を選び、 ``Source`` に ``jubatus_handson`` の ``Group ID`` （下記の例では、sg-6b58146aとなっています。上の表にある自分の環境のGroup IDを参照してください）を入力し、 ``Add Rule`` をクリック

.. image:: http://gyazo.com/cd8a71be8fd716fed6e5e900ef17850b.png

7. さらに下の ``Apply Rule Change`` をクリックして、すべての設定を反映させます。


セキュリティグループとして、 ``jubatus_handson`` を作ります。
インターネットからの接続は22番ポート（SSH）のみを許可し、それ以外はすべて接続を拒否します。一方、 ``jubatus_handson`` グループ内に作られた仮想マシン内ではすべてのTCP通信を許可しています。
jubatusには、アクセスコントロール機能はありません。インターネット上で利用する場合は、必ずこういったセキュリティ設定を行ってください。


ハンズオン用のVMを起動
-------------------------
https://console.aws.amazon.com/ec2/home?region=ap-northeast-1#s=Instances

ハンズオン用のAMIから5台の仮想マシンを起動します。

1. ダッシュボードの左側のメニュー ``INSTANCES`` から ``Instances`` をクリック
2. ``Launch Instance`` をクリック
3. ``Classic Wizard`` を選択、 ``Continue`` をクリック
4. ``Community AMIs`` を選択、Searchの欄に ``jubatus`` を入力
5. Manifestが ``142838215056/jubatus_handson_develop`` となっているものを ``Select``
6. ``Number of Instances:`` に5を入力、 ``Instance Type:`` にM1 Small以上を選択
7. ``Avalialbity Zone:`` あるいは ``Subnet:`` の設定が、デフォルトでは選択しないことになっているので、いずれかの具体的なネットワーク名を指定してください。
8. Contiuneを4回クリック
9. ``Choose from your existing Key Pairs`` で、先ほど作成した、 ``jubatus_handson`` を選択、 ``Continue``
10. ``Choose one or more of your existing Security Groups`` で、先ほど作成した ``jubatus_handson`` を選択、 ``Contiune``
11. すべて揃えば、 ``Launch`` で起動します。

ダッシュボードに戻ってInstancesを見ると、5台のマシンが起動していることがわかると思います。
便宜上、 ``MANAGER`` , ``C1`` , ``C2`` , ``S1`` , ``S2`` と名づけます。NAMEにある鉛筆のアイコンをクリックして、名前をつけましょう。
名前が付けられたら、以下のようになります。

.. image:: http://gyazo.com/25770bc23349e386345eb340a109c543.png

この後、ハンズオンで利用するため、 ``manager`` のプライベートIPアドレスを調べておきます。
``manager`` の行をクリックすると、その情報が下部に表示されます。
画面をスクロールさせて、 ``Private IPs:`` と書かれている所を見てください。
作成した直後の場合、ここが空欄になっている場合がありますが、画面を更新すれば表示されるはずです。
10.X.X.X もしくは 172.X.X.X のようなIPアドレスが書かれているかと思います。これを別の所にメモしておいてください。


managerにssh接続（Windowsの場合）
--------------------------------------------
https://console.aws.amazon.com/ec2/home?region=ap-northeast-1#s=Instances

managerにsshで接続します。先ほど調べたプライベートIPアドレスではなく、グローバルIPアドレスを調べて利用します。
sshクライアントとしてputtyを使用します。puttyではなくCygwin等を用いる場合は、この節でなく、Windows以外の場合の節の説明の通りにしてください。
puttyではopenssh形式であるjubatus_handson.pemをそのまま扱えないので、puttygenというツールで変換して用います。

1. ダッシュボードの左側のメニュー ``INSTANCES`` から ``Instances`` をクリック（VMを立ち上げる操作の直後なら必要ない）
2. ``manager`` を右クリック
3. 表示されたコンテキストメニューの ``Connect`` をクリック
4. ``Connect with a standalone SSH Client`` をクリックすると、4番目にIPアドレスと例が表示される。（例は使わない。）
5. puttyのダウンロードページから ``putty.exe`` と ``puttygen.exe`` をダウンロードする。
6. ``puttygen.exe`` を実行し、 ``File->"Load private key"`` で ``jubatus_handson.pem`` を開く。

.. image:: https://gist.github.com/gwtnb/e5f614edbf58ff9d4ee9/raw/1fc3ceaa7478e584de46cc7143da16b5a25d27a2/puttygen.png

7. ダイアログが開くので ``OK`` をクリックする。
8. ``Save public key`` をクリックして、変換された秘密鍵ファイル ``jubatus_handson.ppk`` を保存する。
9. ``puttygen.exe`` を閉じる。
10. ``putty.exe`` を実行する。
11. ``Category:`` の ``Session`` をクリックし、 ``Host Name (or IP address)`` に4で調べたグローバルIPアドレスを入力する。

.. image:: https://gist.github.com/gwtnb/e5f614edbf58ff9d4ee9/raw/8c82ca13cc01fbf4e7f9d4ad5e4d338ef2168f16/putty_ip.png

12. ``Category:`` の ``Connection/SSH/Auth`` をクリックし、 ``Private key file for authentication`` にjubatus_handson.ppkを指定する。

.. image:: https://gist.github.com/gwtnb/e5f614edbf58ff9d4ee9/raw/e392c8fbf9ba47f68f2b5bb6275868b2f937a80b/putty_key.png

13. ``Open`` をクリックするとコンソールが開き、ユーザー名を問われるので ``ubuntu`` と打つと接続できる。


managerにssh接続（Windows以外の場合）
-------------------------------------------
https://console.aws.amazon.com/ec2/home?region=ap-northeast-1#s=Instances

managerにsshで接続します。先ほど調べたプライベートIPアドレスではなく、グローバルIPアドレスを調べて利用します。

1. ダッシュボードの左側のメニュー ``INSTANCES`` から ``Instances`` をクリック（VMを立ち上げる操作の直後なら必要ない）
2. ``manager`` を右クリック
3. 表示されたコンテキストメニューの ``Connect`` をクリック
4. ``Connect with a standalone SSH Client`` をクリックすると、4番目にIPアドレスと例が表示される。（例は使わない。）
5. ターミナルに ``ssh -i jubatus_handson.pem ubuntu@<4で表示されたグローバルIPアドレス>`` と打つと接続できる。
