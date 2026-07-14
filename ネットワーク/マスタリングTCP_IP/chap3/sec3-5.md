###　公衆アクセス網

- <font color=red><b>公衆通信サービス</b>とは、電話のようにNTTやKDDI、ソフトバンクなどの通信事業者に料金を払って通信回線を借りる形態</font>。

#### アナログ電話回線

- <font color=red>固定電話回線で通信を行う</font>。現在はほとんど利用されていない。
- 特別な通信回線を必要とせず、一般家庭に広く普及している電話網をそのまま利用できる。
- コンピュータをアナログ電話回線で接続するためには、デジタル信号とアナログ信号を変換するモデムが必要。

```plantuml
@startuml
title アナログ電話回線
left to right direction

rectangle 自宅 {
    node 家庭内PC as pc
    node モデム as modem
}
node モデム内蔵のノートPC as pc_modem
rectangle ISPのネットワーク #aaf {
    node "アクセスサーバ/\nルータ" as router1
    node ルータ as router2
    node "メールサーバ/\nWebサーバ" as server
    interface ネットワーク as network
}
cloud インターネット as internet

pc -- modem
modem =[#red]= router1 : <color red>電話回線で\n<color red>PPPを使用
pc_modem -- router1
network - router1
network -- router2
server - network
router2 - internet
@enduml
```

#### ADSL(Asymmetric Digital Subscriber Line)

- <font color=red>既存のアナログ電話回線を拡張利用するサービス</font>。ADSLでは、電話機と電話局の交換機の間の回線にスプリッタと呼ばれる分配器を設置し、音声周波数(低周波)とデータ通信用の周波数(高周波)を混合・分離する。
- 近年の電話網はデジタル化により周波数がカットされ64kbps程度のデジタル信号に変換されてしまうため、64kbps以上の速度で通信するのは原理的に不可能。
- ADSL以外にも、VDSL、HDSL、SDSLなどがあり、総称してxDSLと呼ぶ。ADSLはその中で最も普及している方式である。

```plantuml
@startuml
title ADSL接続
left to right direction

rectangle 家庭 {
    node 電話 as landline
    node "P C" as pc
    rectangle ADSLモデム as modem1
    rectangle スプリッタ as spliter1
}
note bottom of spliter1
回線速度は通信方式や電話回線の品質、
電話局からの距離などによって異なるが、
ISP→家庭・オフィスが1.5Mbps〜50Mbps、
家庭・オフィス→ISPが512kbps〜2Mbps程度。
end note
rectangle 電話局 {
    rectangle スプリッタ as spliter2
    rectangle 交換機 as exchanger
    rectangle ADSLモデム as modem2
}
rectangle ISP
cloud インターネット as internet
queue デジタル回線 as digital_line

landline -- spliter1
pc = modem1 : デジタル
modem1 - spliter1
spliter1 == spliter2 : 【アナログ回線】\n低周波と高周波を\n混合・分離し高速化する。
spliter2 - exchanger
exchanger == digital_line : デジタル回線\n(64kps)
modem2 - spliter2
modem2 <=[#blue]= ISP : <color blue>デジタル回線\n<color blue>(1.5Mbps〜50Mbps)
modem2 =[#red]=> ISP : <color red>デジタル回線\n<color red>(512kbps〜2Mbps)
ISP - internet
@enduml
```

<div style="page-break-before:always"></div>

#### FTTH(Fiber To The Home)

- <font color=red>高速の光ファイバをユーザの自宅や会社の建物内に直接引き込む手法</font>
- 建物までは光ファイバ、建物からはONUを経由してルータやコンピュータに接続する。
- **敷設された光ファイバー回線で光信号が通っておらず、未使用の芯線をダークファイバーという**。光ファイバーの敷設工事には莫大なコストがかかるため、あらかじめ多めに光ファイバーを設置しており、その予備の光ファイバーをダークファイバーとしている。

```plantuml
@startuml
title FTTH

rectangle 建物 {
    node "P C" as pc
    rectangle 無線ルータ as router1
    rectangle ONU
}
note bottom of ONU
Optical Network Unit
光回線終端装置
end note
rectangle 光スプリッタ as spliter1
note bottom of spliter1
光信号を合分波するための機器。
より安価な光回線を提供することが可能。
end note
rectangle 通信会社のネットワーク {
    rectangle 光スプリッタ as spliter2
    rectangle OLT
    rectangle ルータ as router2
}
note bottom of OLT
Optical Line Terminal
通信会社側の光回線終端装置
end note
cloud インターネット as internet

pc -- router1
router1 - ONU
spliter1 -- ONU
spliter1 =[#red] spliter2 : <color red>光回線
spliter2 - OLT
OLT - router2
internet -- router2
@enduml
```

#### ケーブルテレビ

- <font color=red>電波を使うテレビ方法をケーブルを使って放送するサービス</font>
- 電波による地上放送はアンテナの設置状況や周りの建物によって受信状態が悪くなる可能性があるが、ケーブルはその影響が少ない。
- 近年、空いているチャネルをデータ通信専用に利用するケーブルテレビを使ったインターネット接続サービスが広く行われるようになった。
- ダウンストリーム（放送局から加入者宅までの通信）はテレビ放送と同じ周波数帯を使用し、アップストリーム（加入者宅から放送局までの通信）は放送では利用されていない低周波数帯を使用。

```plantuml
@startuml
title ケーブルテレビ
left to right direction

rectangle 家庭 {
    rectangle テレビ as television
    rectangle "P C" as pc
    rectangle "データ通信用の\nケーブルモデム" as modem
}
rectangle ケーブルテレビ局 {
    rectangle ヘッドエンド as headend
    rectangle テレビ放送 as broadcast
}
note top of headend
デジタル放送や一部のアナログ放送と
通信用のデジタルデータを
1つのケーブルで送受信できるように
変換する機器。
end note
rectangle ISP
interface アンテナ as antenna
queue 他局のテレビ放送 as other_broadcast
cloud インターネット as internet

pc -- modem
television -- headend
modem =[#red]=> headend : <color red>上り（速い）
modem <=[#blue]= headend : <color blue>下り（遅い）
headend - broadcast
broadcast -- antenna
antenna -- other_broadcast
headend -- ISP
ISP -- internet
@enduml
```

#### 専用回線（専用線）

- <font color=red>拠点間を物理的または論理的に1対1で接続する回線サービス</font>。ISDNやフレームリレーのように1回線を引けば数カ所と接続が可能になるわけではない。
- イーサネット専用線が主で1Mbps〜100Gbpsのサービスが提供されている。

<div style="page-break-before:always"></div>

#### VPN(Virtual Private Network)

- <font color=red>VPNは公衆回線上に仮想的なプライベートネットワークを設けること</font>。
- IP-VPNや広域イーサネット、SD-WANサービスなどがある。
- **IP-VPN(ネットワーク層)**: IPネットワーク(インターネット)にVPNを構築したもの。①通信事業者が提供するIP-VPNサービスと②企業独自のVPN(インターネットVPN)がある。
  - 通信事業者が提供するものとしてIPネットワーク上にMPLS(MultiProtocol Label Switching)技術を用いてVPNを構築する手法がある。MPLSはIPパケットにラベルを付与することで、複数の顧客のVPNを1つのMPLS網上で区別する仕組み。顧客ごとに帯域補償などを行うことが可能。
  - インターネットVPNはIPsecを使ってVPNを実現する方法が一般的。IPsecを用いてVPN上での通信時にIPパケットの認証、暗号化を行い閉じたネットワークを構築する。IPsecは安価な上、各自が必要とするセキュリティレベルを設定できる利点があるが、混雑具合によって通信速度に影響が出る欠点もある。
- **広域イーサネット(データリンク層)**: 通信事業者が提供する離れた地域を結ぶイーサネット接続のサービス。広域イーサネットはデータリンク層のVLANを利用する。IP-VPNと異なり、TCP/IP以外のプロトコルも利用できる。
  - 通信事業者が構築するネットワークのVLANを利用企業が専用で利用する形になる。
  - 広域イーサネットはデータリンク層を利用しているため、不要なパケットを流さないように定期的にメンテナンスし、利用者が工夫した運用をする必要がある。
- **SD-WANサービス**: WANを構成するMPLSやインターネット、4G LTEを取りまとめ、仮想的なWANリンクを構成するサービス。論理ネットワークを構成することができ、経路の暗号化や経路制御などの機能が提供されることもある。

```plantuml
@startuml
title "IP-VPN ( MPLS )"
left to right direction

rectangle ネットワーク1 {
    rectangle "P C" as pc1
    rectangle "P C" as pc2
    rectangle ルータ as router1
    pc1 -- router1
    pc2 -- router1
}
cloud "IP-VPN\n( MPLS )" as ipvpn #aaf {
    rectangle ルータ as router2
    rectangle ルータ as router3
    router2 == router3
}
note right of ipvpn
仮想的な
独自ネットワークを
構築できる。
ラベルを用いて
複数の顧客のVPNを
1つのMPLS網上で
区別できる。
end note
rectangle ネットワーク2 {
    rectangle "P C" as pc3
    rectangle "P C" as pc4
    rectangle ルータ as router4
    router4 -- pc3
    router4 -- pc4
}

router1 == router2 : ラベル付与\n(タグ付与)
router3 == router4 : ラベル除去\n(タグ除去)
@enduml
```

#### 公衆無線LAN

- <font color=red>Wi-Fi(IEEE802.11bなど)を利用したサービス</font>
- 電波受信可能エリア(ホットスポット)を駅や飲食店などに設置し、ユーザはホットスポット経由でインターネットに接続する。接続後、IPsecを利用したVPN経由で自身の会社へ接続も可能。
- 公衆無線LANは無料の場合と有料の場合があり、セキュリティの有無を確認する必要がある。

```plantuml
@startuml
title 公衆無線LAN
left to right direction

cloud インターネット as internet
interface " " as interface1
interface " " as interface2
storage 無線LAN as lan1 {
    rectangle "P C" as pc1
    node ホットスポット as hotspot1
}
storage 無線LAN as lan2 {
    rectangle "P C" as pc2
    node ホットスポット as hotspot2
}
storage 無線LAN as lan3 {
    rectangle "P C" as pc3    
    node ホットスポット as hotspot3
}
storage 無線LAN as lan4 {
    rectangle "P C" as pc4
    node ホットスポット as hotspot4
}

pc1 -- hotspot1
hotspot2 - pc2
hotspot3 - pc3
pc4 - hotspot4
internet - interface1
internet -- interface2
internet -up- hotspot3
hotspot1 -- interface1
interface1 -- hotspot4
interface2 -- hotspot2
@enduml
```
