AWSの設定
===============

ハンズオンを始める前に、Amazon web serviceにsign inしてください。以降、すべてAWSに接続されている状態であることを想定します。これから、ハンズオンを実施するにあたって必要な設定を行います。既にAWSを使いこなしている方は、適時設定を飛ばす、読み替える、などしていただいて構いません。

- クーポンによるチャージ：今回のハンズオンでは、アマゾン・ジャパン様のご厚意により、ハンズオンを実施するために十分なクーポンをいただいております。これをチャージして利用します。
- Key Pairの作成：今回のハンズオンでは、今日のハンズオン専用のAMI(Amazon Machine Image)を用います。この仮想マシンにログインするための鍵ペアを作成します。
- Security Groupsの作成、設定：今回のハンズオンは分散環境で動作します。同じ環境同士の通信は自由にできるように、かつ他の環境からは通信できないようにセキュリティグループを設定します。
- ハンズオン用のVMを起動：ハンズオン用のAMIから5台の仮想マシンを起動します。便宜上、"MANAGER", "C1", "C2", "S1", "S2"と名づけます。
- VMへ接続：putty, TeraTerm, などSSHクライアントを用いて、環境に接続します。
 


クーポンによるチャージ
-------------------------
https://portal.aws.amazon.com/gp/aws/manageYourAccount

1. 上部、アカウント／コンソールから"アカウント"をクリック
2. 左のメニューから"支払い方法"をクリック
3. 右画面下部のAWS割引クーポンを引き換える、にある"AWSクレジットを引き換え／表示" をクリック
4. 下記のクレームコードを入力し、引き換えをクリックしてください。AWSアカウントに支払いを追加します。 から指定されたコードを入力。
5. ただしくチャージされていれば、次のようになります。

![payment](https://gist.github.com/odasatoshi/d95e62b070c1a679abd4/raw/e04c026e4a696c3345cda31c820436518bc9ff36/payments.png)


EC2 ダッシュボード
------------------------
https://console.aws.amazon.com/ec2/v2/home

1. 上部、アカウント／コンソールから"AWS Management Console"
2. EC2ダッシュボートをクリック

ここで、URLの"region"が、"ap-northeast"となっていることを確認して下さい。
後ろに数字が付いているのは構いませんが、例えば"us-west"などになっていると、
以下のチュートリアルが実施できないことがあります。
右上の"あなたの名前"と"Help"の間にある地名が、"Tokyo"になっていれば大丈夫です。
![dashborad](https://gist.github.com/odasatoshi/d95e62b070c1a679abd4/raw/f03d68723ddcbc49c2d5f98fd1da860fac296d38/dashboard.png)

Key Pairの作成
------------------------
https://console.aws.amazon.com/ec2/home?region=ap-northeast-1#s=KeyPairs

1. ダッシュボードのメニュー "NETWORK & SECURITY" から、"Key Pairs"をクリック
2. Create Key Pairボタンをクリック
3. Key Pair Nameに"jubatus_handson"と入力して"Create"ボタンをクリック
4. jubatus_handson.pemというファイルがダウンロードされます。

このpemファイルはこれから作るサーバに接続するために必要な秘密鍵です。適時適切な所に配置してください。
Linux, Macなどをお使いの方は、ファイルPermissionの設定をする必要があります。具体的には

    chmod 600 jubatus_handson.pem

を実行しておかないと、sshクライアントが受け付けてくれません。
鍵は既にあるものを利用しても構いません。適時読み替えてください。

Security Groupsの作成、設定
-----------------------------
https://console.aws.amazon.com/ec2/home?region=ap-northeast-1#s=SecurityGroups

1. ダッシュボードのメニュー "NETWORK & SECURITY" から、"Security Groups"をクリック
2. Create Security Groupボタンをクリック
3. Nameに"jubatus_handson", Descriptionに適当な説明を入力して"Yes, Create"ボタンをクリック
4. 作成したグループを選択し、下部の"Inbound"をクリック
5. Port Rangeに"22"を追加して、下のボタン、Add Ruleをクリック
![dashborad](https://gist.github.com/odasatoshi/d95e62b070c1a679abd4/raw/b7cec527ba2d042c00a1f40bbc8c4871b147999a/secgroup.png)
6. Create a new ruleから、All TCPを選び、Sourceに"jubatus_handson"のGroup ID（下記の例では、sg-6b58146aとなっています。上の表にある自分の環境のGroup IDを参照してください）
![custom](http://gyazo.com/cd8a71be8fd716fed6e5e900ef17850b.png)
7. さらに下のApply Rule Changeをクリックして、すべての設定を反映させます。


セキュリティグループとして、jubatus_handsonを作ります。
インターネットからの接続は22番ポート（SSH）のみを許可し、それ以外はすべて接続を拒否します。一方、"jubatus_handson"グループ内に作られた仮想マシン内ではすべてのTCP通信を許可しています。
jubatusには、アクセスコントロール機能はありません。インターネット上で利用する場合は、必ずこういったセキュリティ設定を行ってください。


ハンズオン用のVMを起動
-------------------------
https://console.aws.amazon.com/ec2/home?region=ap-northeast-1#s=Instances

ハンズオン用のAMIから5台の仮想マシンを起動します。

1. ダッシュボードのメニュー "INSTANCES"から"Instances"をクリック
2. "Launch Instance"をクリック
3. "Classic Wizard"を選択、"Continue"をクリック
4. "Community AMIs"を選択、Searchの欄に"jubatus"を入力
5. Manifestが"142838215056/jubatus_distributed_handson"となっているものを"Select"
6. Number of Instances:に5を入力、Instance Type:にM1 Small以上を選択、Availability Zone:を適当な所に設定
7. Contiuneを4回クリック
8. Choose from your existing Key Pairsで、先ほど作成した、"jubatus_handson"を選択、Continue
9. Choose one or more of your existing Security Groupsで、先ほど作成した"jubatus_handson"を選択、Contiune
10. すべて揃えば、Launchで起動します。

ダッシュボードに戻ってInstancesを見ると、5台のマシンが起動していることがわかると思います。
便宜上、"MANAGER", "C1", "C2", "S1", "S2"と名づけます。NAMEにある鉛筆のアイコンをクリックして、名前をつけましょう。
名前が付けられたら、以下のようになります。
![servers](http://gyazo.com/25770bc23349e386345eb340a109c543.png)

この後、ハンズオンで利用するため、managerのプライベートIPアドレスを調べておきます。
managerの行をクリックすると、その情報が下部に表示されます。
画面をスクロールさせて、"Private IPs:" と書かれている所を見てください。
作成した直後の場合、ここが空欄になっている場合がありますが、画面を更新すれば表示されるはずです。
10.X.X.X もしくは 172.X.X.X のようなIPアドレスが書かれているかと思います。これを別の所にメモしておいてください。

VMへ接続
--------

    マシン, セッション数
    manager, 4
    c1, 3
    c2, 2
    s1, 1
    s2, 1


