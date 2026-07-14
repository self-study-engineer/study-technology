###　通信方式の種類

- データの配送方法は**コネクション型**と**コネクションレス型**に分けられる。
- 通信対象の数による通信方式の分類は**ユニキャスト**、**ブロードキャスト**、**マルチキャスト**、**エニーキャスト**がある。
- 通信方法は**回線交換方式**と**パケット交換方式**がある。

```plantuml
@startuml

left to right direction
rectangle delivery_type [
  <b>データ配送方式(2通り)
  ・コネクション型
  ・コネクションレス型
]
rectangle cast_type [
  <b>通信対象の数による通信方式(4通り)
  ・ユニキャスト(従来の電話)
  ・ブロードキャスト(テレビ放送)
  ・マルチキャスト(ビデオ会議)
  ・エニーキャスト(DNSのルートネームサーバ)
]
rectangle communication_type [
  <b>通信方式(2通り)
  ・回線交換方式
  ・パケット交換方式
]
label "✖️" as times1
label "✖️" as times2

delivery_type -[hidden]- times1
times1 -[hidden]- cast_type
cast_type -[hidden]- times2
times2 -[hidden]- communication_type

@enduml
```

#### コネクション型とコネクションレス型

| 比較項目             | コネクション型 (Connection-oriented)                   | コネクションレス型 (Connectionless)                         |
|----------------------|------------------------------------------------------|-----------------------------------------------------------|
| 例                   | TCP (Transmission Control Protocol)、ATM、フレームリレー | UDP (User Datagram Protocol)、イーサネット、IP |
| 接続の確立           | <font color=red>データ転送前に通信相手との接続を確立する必要がある。</font>       | <font color=red>接続の確立は不要。データを直接送信できる。</font>                    |
| 信頼性               | 高い。パケットの順序確認、再送制御、エラー検出を行うが、遅延が発生する可能性がある   | 低い。パケット送信だけであり遅延はないが、順序保証や再送制御はなく、エラー検出も限定的 |
| データ転送の順序     | パケットの順序を保証する                                | 順序は保証されない                                         |
| 再送制御             | パケットが欠落した場合、自動的に再送される               | 再送制御は行われない                                       |
| 適した用途           | 信頼性が重要なアプリケーション (例：ファイル転送、メール) | リアルタイム性が重要なアプリケーション (例：VoIP、ゲーム)    |

#### ユニキャスト/マルチキャスト/ブロードキャスト/エニーキャスト

```plantuml
@startuml
rectangle "ユニキャスト\n(1対1の通信)" as uni_cast {
  node PC as pc1
  node PC as pc2
  node PC as pc3
  node PC as pc4

  pc1 --> pc2
  pc1 -[hidden]-> pc3
  pc1 -[hidden]-> pc4
}
rectangle "ブロードキャスト\n(同じデータリンク内の全てのコンピュータ)" as broad_cast {
  node PC as pc5
  node PC as pc6
  node PC as pc7
  node PC as pc8

  pc5 --> pc6
  pc5 --> pc7
  pc5 --> pc8
}
rectangle "マルチキャスト\n(特定のグループ内の通信)" as multi_cast {
  node PC as pc9
  node PC as pc10
  node PC as pc11
  node PC as pc12

  pc9 --> pc10
  pc9 --> pc11
  pc9 -[hidden]- pc12
}
rectangle "エニーキャスト\n(特定のグループの任意の1つ)" as any_cast {
  node PC as pc13
  node PC as pc14
  node PC as pc15
  node PC as pc16

  pc13 ..> pc14
  pc13 ..> pc15
  pc13 ..> pc16
}

uni_cast -[hidden]-- multi_cast
broad_cast -[hidden]-- any_cast
@enduml
```

#### 回線交換方式とパケット交換方式

```plantuml
@startuml
title "回線交換方式(【例】同時に2回線、2ユーザのみ通信可能)"
left to right direction

node PC as pc1
node PC as pc2
node PC as pc3
node PC as pc4
node PC as pc5
node PC as pc6
rectangle 回線交換機 as circuit_switch1 {
  interface " " as i1
  interface " " as i2
  interface " " as i3
  interface " " as i4
  interface " " as i5
}
rectangle 回線交換機 as circuit_switch2 {
  interface " " as i6
  interface " " as i7
  interface " " as i8
  interface " " as i9
  interface " " as i10
}

pc1 -- i1
pc2 -- i2
pc3 -- i3
i6 -- pc4
i7 -- pc5
i8 -- pc6
i1 -- i4
i3 -- i5
i9 -- i6
i10 -[hidden]- i7
i10 -- i8
i4 -[#red,thickness=3]- i9 : <color red>回線1
i5 -[#red,thickness=3]- i10 : <color red>回線2

@enduml
```
```plantuml
@startuml
title "パケット交換方式(【例】1つの回線を3つのバッファで共有)"
left to right direction

node PC as pc1
node PC as pc2
node PC as pc3
node PC as pc4
node PC as pc5
node PC as pc6
rectangle パケット交換機 as packet_switch1 {
  interface "バッファ" as b1
  interface "バッファ " as b2
  interface "バッファ" as b3
  interface "キュー" as q1
}
rectangle パケット交換機 as packet_switch2 {
  interface "バッファ" as b4
  interface "バッファ " as b5
  interface "バッファ" as b6
  interface "キュー" as q2
}

pc1 -- b1
pc2 -- b2
pc3 -- b3
b4 -- pc4
b5 -- pc5
b6 -- pc6
b1 -- q1
b2 -- q1
b3 -- q1
q2 -- b4
q2 -- b5
q2 -- b6
q1 -[#red,thickness=3]- q2 : <color red>1回線(共有)

@enduml
```

| 比較項目             | 回線交換方式 (Circuit Switching)                        | パケット交換方式 (Packet Switching)                     |
|----------------------|--------------------------------------------------------|---------------------------------------------------------|
| 例                   | 電話網 (旧式の固定電話システム)                          | インターネット、LAN                                      |
| データ転送の順序     | データは送信された順番通りに到着する                     | パケットは経路ごとに異なる順序で到着する可能性がある      |
| 遅延                 | 一度回線が確立されると遅延は低い                         | パケットが異なるルートを通るため、遅延が変動することがある  |
| 帯域幅の効率 | 非効率的。<font color=red>回線が専有されている間、データを送信していない場合でもリソースが占有される。</font> | 効率的。<font color=red>リソースは必要に応じて動的に共有される。</font> |
| 信頼性               | 高い。回線が確立されると、データの損失は少ない           | パケットの再送やエラー検出により信頼性を確保             |
| 適した用途           | 連続的で一定の帯域幅を必要とする通信 (例：音声通話)       | 帯域幅が変動し、効率的なリソース使用が求められる通信 (例：データ通信、インターネット) |