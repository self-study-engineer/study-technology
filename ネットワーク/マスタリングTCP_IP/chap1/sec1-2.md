###　コンピュータとネットワーク発展の7つの段階
#### バッチ処理(Batch Proccessing, 1950年代)

- <font color=red>当時のコンピュータは高価で巨大なモノで誰もが気軽に使えるモノではなかった</font>。
- プログラム実行やデータ操作する場合、計算機センターまで行かなければならなかった。
- プログラムの実行は専門のオペレータに依頼し、後日、処理結果を計算機センターまで撮りに行く必要があった。

```puml
@startuml
title バッチ処理

actor ユーザ as user

rectangle 計算機センター {
    actor オペレータ as operator
    file "プログラム(カード)" as program
    rectangle カードリーダ as card_reader
    node コンピュータ as computer #aaf
    node プリンタ as printer
}

user -> program : 作成
program -> operator : 渡す
operator --> card_reader : ①記録
card_reader -> computer : ②逐次実行
computer -> printer : ③反映

@enduml
```

#### タイムシェアリングシステム(TSS: Time Sharing System, 1960年代)

- 仮想的に一人の人が1台のコンピュータを占有して利用する事が可能
- インタラクティブ(対話的)な操作が可能になった。
- **スター型**(ホストコンピュータを中心に端末が接続された通信形態)で接続
- <font color=red>コンピュータとコンピュータを繋いでいるわけではない</font>。

```puml
@startuml
title タイムシェアリングシステム

node ホストコンピュータ as host #aaf
note right of host
**マルチタスク方式**
デバイスドライバを含むOSが
複数のプログラムを
短い時間で切り替えながらCPUに
処理させる方式
end note
rectangle 端末 as monitor1
rectangle 端末 as monitor2
rectangle 端末 as monitor3
rectangle 端末 as monitor4
note bottom of monitor1
キーボードとディスプレイを
備えた入出力装置。
初期の頃は
タイプライターが利用された。
end note

host <-- monitor1
host <-- monitor2
host <-- monitor3
host <-- monitor4

@enduml
```

<div style="page-break-before:always"></div>

#### コンピュータ間通信(1970年代)

- コンピュータ性能が飛躍的に向上し、安価になった。
- データ転送にかかる時間が一気に少なくなった（物理的移動から通信での移動に変わった）。
- <font color=red>利用者の目的や規模に合わせた柔軟なシステムの構築や運用ができるようになった</font>。

```puml
@startuml
title コンピュータ間通信

node 業務A用コンピュータ as hostA #aaf
rectangle 端末 as monitor1
rectangle 端末 as monitor2
rectangle 端末 as monitor3
note bottom of monitor1
キーボードとディスプレイを
備えた入出力装置。
初期の頃は
タイプライターが利用された。
end note
node 業務B用コンピュータ as hostB #aaf
rectangle 端末 as monitor4
rectangle プリンタ as printer1
node 業務C用コンピュータ as hostC #aaf
rectangle 端末 as monitor5
rectangle プリンタ as printer2

hostA <-- monitor1
hostA <-- monitor2
hostA <-- monitor3
hostB <-- monitor4
hostB <-- printer1
hostC <-- monitor5
hostC <-- printer2
hostA -[thickness=3] hostB
hostB -[thickness=3] hostC

@enduml
```

#### コンピュータネットワーク(1980年代)

- 異なるメーカ同士の多種多様なコンピュータがネットワークに結ばれるようになった。
- 画面上で複数の窓(ウィンドウ)を開く事ができるシステムであるウィンドウシステムが登場した。
- <font color=red>コンピュータの発展と普及がネットワークをより身近なモノにした</font>。

```puml
@startuml
title コンピュータネットワーク
left to right direction

interface "トークンリング" as token_ring
note bottom of token_ring
LANの物理層および
データリンク層の規格の一つ。
IBMが開発したもの。
end note
rectangle ルータ as router1 #aaf
node PC as pc1
node PC as pc2
rectangle ルータ as router2 #aaf
node PC as pc3
rectangle ルータ as router3 #aaf
rectangle ルータ as router4 #aaf
node PC as pc4
node PC as pc5
rectangle ルータ as router5 #aaf
rectangle ハブ as hub1 #afa
node PC as pc6
node PC as pc7
node PC as pc8
node PC as pc9
node PC as pc10

pc1 -- router1
pc2 -- router1
router1 -- router2
pc3 - router2
router2 - router3
router2 -- router4
router4 -[#red,thickness=2]- router5 : <color red>専用回線
router5 -- pc4
router5 -- pc5
router3 -- hub1
token_ring -- router3
pc6 -- token_ring
pc7 -- token_ring
pc8 -- token_ring
router3 - pc9
hub1 -- pc10

@enduml
```

<div style="page-break-before:always"></div>

#### インターネットの普及(1990年代)

- マルチベンダ接続やダウンサイジング(コンピュータが性能向上しつつ安価になった動き)によりシステム構築が容易になった。
- <font color=red>企業も一般家庭もインターネットに接続するようになった。</font>

```puml
@startuml
title インターネットの普及
left to right direction

rectangle 全世界 {
    storage A社ネットワーク #ddd {
        rectangle ルータ as router #aaf
        rectangle ハブ as hub #afa
        node PC as pc1
        node PC as pc2
        node サーバ as server
        node 外部向けサーバ as ext_server
        node 認証サーバ as auth_server
    }
    cloud インターネット as internet
    rectangle "<b>A社と関係のある会社・組織・一般家庭\n認証サーバで認証後、\nA社内のサーバにもアクセス可能。" as business_partner
    rectangle "<b>A社と関係のない会社・組織・一般家庭\nA社内のサーバにはアクセスできないが\n外部向けサーバから情報取得可能。" as others
}

pc1 -- hub
pc2 -- hub
server -- hub
hub -- router
router -- ext_server
auth_server - router
router - internet
internet -- business_partner
internet -- others

@enduml
```

#### いつでもどこでも何にでもTCP/IPネットワークの時代

- **汎用通信基盤が電話網からIP網へ変わった**。
- ネットワークに繋がる機器がコンピュータだけでなく、携帯端末や家電製品、ゲーム機などに広がった。
- これまで、**外部と接続しない閉域網**として制御系システムが構築されていたが、インターネット接続が増えてきた。
- <font color=red>ありとあらゆるものがインターネットにつながるようになった</font>

```puml
@startuml
title IPによる通信・放送の統一(電話網からIP網へ)
left to right direction

cloud "IPネットワーク\nW W W\n携帯電話網\nVoIP\niSCSI" as network #aaf
node PC as pc
node 携帯電話 as mobile_phone
node 固定電話 as landing_phone
node カメラ as camera
node テレビ as tv
node 家電 as appliance

pc -- network
mobile_phone -- network
landing_phone -- network
network -- tv
network -- camera
network -- appliance

@enduml
```

#### 「単に繋ぐ」時代から「安全に繋ぐ」時代へ

- 利便性の向上に伴い、情報漏洩や詐欺事件などのトラブルが増加し、企業や個人の活動に大きな損失を与えるようになった。
- 通信の仕組みを理解し、安全で健全な通信手段を維持する事が不可欠な時代になった。
- インターネットは別々に発達してきた多種多様な通信技術を組み合わせたものであり、TCP/IPはインターネットを実現するだけの応用力がある。
- <font color=red>色々な「モノ」に接続し、新たな「コト」を想像する仕組み(<b>IoT: Internet of Things</b>)が増えてきた</font>。
※製造現場においては特に<b>Industrial IoT(Industry4.0)</b>と呼ばれる。
