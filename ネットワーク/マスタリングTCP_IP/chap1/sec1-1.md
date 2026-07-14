###　コンピュータネットワーク登場の背景

- **スタンドアローン**： コンピュータをネットワークに接続せず、単独で使用する状態。<font color=red>それぞれの端末が独立してデータを持ち、修正の際はそれぞれの端末に対して操作</font>する必要がある。
- **コンピュータネットワーク**： コンピュータとコンピュータを接続し、複数のコンピュータを互いに接続して使用できる状態。
  - **LAN**： フロア内や1つの建物の中、キャンパスの中など、比較的狭い地域の中でのネットワーク。
  - **WAN**： 地理的に離れた広範囲に及ぶネットワーク。

```puml
@startuml
title スタンドアローン

rectangle "業務1用端末" as pc_a
rectangle "業務2用端末" as pc_b
rectangle "業務3用端末" as pc_c

actor "ユーザ1" as user1
actor "ユーザ2" as user2
note right of user2
ユーザ1が
終了するまで
待機中
end note
actor "ユーザ3" as user3
note right of user3
ユーザ1、ユーザ2が
終了するまで待機中
end note

pc_a <-- user1 : 利用
pc_b <-- user1 : 利用
pc_c <-- user1 : 利用
pc_c <-[hidden]- user2
pc_c <-[hidden]- user3
user1 -[hidden] user3
@enduml
```

```puml
@startuml
title ネットワークでのコンピュータの利用



rectangle "業務1〜3用サーバ" as server
rectangle "ユーザ1用PC" as pc_1
rectangle "ユーザ2用PC" as pc_2
rectangle "ユーザ3用PC" as pc_3
actor "ユーザ1" as user1
actor "ユーザ2" as user2
actor "ユーザ3" as user3
queue "　　　　　　　　　　ネットワーク　　　　　　　　　"  as network
note right of network
各自が自分一人で
それぞれのコンピュータを利用でき、
業務1〜3を自由に切り替える。
また共通利用するデータはサーバで
一元管理することができる。
end note
server -- network
network -- pc_1
network -- pc_2
network -- pc_3
pc_1 -- user1
pc_2 -- user2
pc_3 -- user3

@enduml
```

```puml
@startuml
title LAN
left to right direction

storage "LAN" as lan {
    interface ルータ as router #aaf
    rectangle スイッチ1 as switch1
    rectangle スイッチ2 as switch2
    rectangle スイッチ3 as switch3
    rectangle PC1 as pc1
    rectangle PC2 as pc2
    rectangle PC3 as pc3
    rectangle PC4 as pc4
    rectangle PC5 as pc5
    pc1 - switch1
    switch1 - pc2
    pc3 - switch2
    switch3 - pc4
    pc5 - switch3
    switch1 -- router
    switch2 - router
    router -- switch3
}
note right of lan
フロア内や1つの建物の中、
キャンパスの中など、
<color red>比較的狭い地域の中でのネットワーク
end note

@enduml
```

```puml
@startuml
title WAN

storage "LAN(福岡)" as lan_fukuoka {
    interface ルータ as router_fukuoka #aaf
    rectangle スイッチ1 as switch_fukuoka
    rectangle PC1 as pc1_fukuoka
    rectangle PC2 as pc2_fukuoka
    lan_fukuoka -- router_fukuoka
    router_fukuoka - switch_fukuoka
    switch_fukuoka -- pc1_fukuoka
    switch_fukuoka -- pc2_fukuoka
}
storage "LAN(大阪)" as lan_osaka {
    interface ルータ as router_osaka #aaf
    rectangle スイッチ1 as switch_osaka
    rectangle PC1 as pc1_osaka
    rectangle PC2 as pc2_osaka
    lan_osaka -- router_osaka
    switch_osaka - router_osaka
    switch_osaka -- pc1_osaka
    switch_osaka -- pc2_osaka
}
storage "LAN(東京)" as lan_tokyo {
    interface ルータ as router_tokyo #aaf
    rectangle スイッチ1 as switch_tokyo
    rectangle PC1 as pc1_tokyo
    rectangle PC2 as pc2_tokyo
    lan_tokyo -- router_tokyo
    switch_tokyo - router_tokyo
    switch_tokyo -- pc1_tokyo
    switch_tokyo -- pc2_tokyo
}
note top of lan_fukuoka
<color red>地理的に</color>離れた広範囲に及ぶネットワーク。
WANよりも狭い都市レベルのネットワークを
MAN(MetropolitanAreaNetwork)と呼ぶ場合もある。
end note
storage "LAN(ロサンゼルス)" as lan_LosAngeles {
    interface ルータ as router_LosAngeles #aaf
    rectangle スイッチ1 as switch_LosAngeles
    rectangle PC1 as pc1_LosAngeles
    rectangle PC2 as pc2_LosAngeles
    lan_LosAngeles -- router_LosAngeles
    router_LosAngeles - switch_LosAngeles
    switch_LosAngeles -- pc1_LosAngeles
    switch_LosAngeles -- pc2_LosAngeles
}
lan_fukuoka -[thickness=3] lan_osaka
lan_fukuoka -[thickness=3]- lan_tokyo
lan_osaka -[thickness=3]- lan_tokyo
lan_tokyo -[thickness=3]- lan_LosAngeles

@enduml
```