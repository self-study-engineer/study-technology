###　アドレスとは

- アドレスはMACアドレスやIPアドレス、ポート番号などが挙げられる。
- アドレスには唯一性と階層性がある。
- MACアドレスは<b>転送表（フォワーディングテーブル）</b>からネットワークインタフェースを決定する。転送表にはMACアドレスがそのまま記録される。
- IPアドレスは<b>経路制御表（ルーティングテーブル）</b>からネットワークインタフェースを決定する。経路制御表にはIPアドレスのネットワーク部とサブネットマスクが記録される。

|  | MACアドレス | IPアドレス |
| -- | -- | -- |
| 唯一性 | ある | ある |
| 階層性 | ない | ある |

```plantuml
@startuml
title 転送表と経路制御表によるパケット送出先の決定

cloud ネットワーク1 {
    node ルータ1 as router1
    storage スイッチ1 as switch1
    rectangle "ホストA" as pc1
    rectangle "ホストB" as pc2
    
}
cloud ネットワーク2 {
    node ルータ2 as router2
    storage スイッチ2 as switch2
    rectangle "ホスト1" as pc3
    rectangle "ホスト2" as pc4
}
cloud ネットワーク3 {
    node ルータ3 as router3
    storage スイッチ3 as switch3
    rectangle "ホストα" as pc5
    rectangle "ホストβ" as pc6
}
note right of switch3
⑤
**転送表**に従って、
ホストβに転送する。
end note
note right of router3
④
**経路制御表**に従って、
スイッチ3宛に送り出す
end note
note top of router1
③
**経路制御表**に従って、
ルータ3宛に送り出す
end note
note top of switch1
②
**転送表**に従って、
ルータ1宛に送り出す
end note
note bottom of pc1
①
自分の**経路制御表**を見て
ホストβ宛のデータを
ルータ1宛に送り出す
end note

switch1 <-[#red,thickness=3]- pc1 : <color red>①
router1 <-[#red,thickness=3]- switch1 : <color red>②
router1 -[#red,thickness=3]> router3 : <color red>③
router3 -[#red,thickness=3]-> switch3 : <color red>④
switch3 -[#red,thickness=3]-> pc6 : <color red>⑤

router2 -- router1
router2 -- router3
router2 - switch2
switch2 - pc3
pc4 -- switch2
switch1 -- pc2
switch3 -- pc5

@enduml
```