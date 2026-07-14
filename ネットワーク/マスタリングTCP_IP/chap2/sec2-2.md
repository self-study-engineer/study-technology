###　TCP/IPの階層モデル

- OSI参照モデルは「通信プロトコルに必要な機能は何か」を中心に考えており、<b>理論的で標準的なモデル化</b>がされている。
- <font color=red>TCP/IPの階層モデルは「プロトコルをコンピュータに実装するにはどのように実装したら良いのか」を中心に考えており、<b>実践的なモデル化</b>がされている</font>。
- **第一層のハードウェア**は特に制約はなく、ネットワークで接続された装置間で通信できることを前提にして作られれている。
- **第二層のネットワークインタフェース層**は、NICを動かすための「デバイスドライバ」。
- **第三層のインターネット層**は、最終目的地のホストまでパケットを届ける役割を持つ。
- **第四層のトランスポート層**は、アプリケーションプログラム間の通信を実現する役割を持つ。
- **第五層のアプリケーション層**は、データの送信や取り扱い、通信手順や方法について実現する役割を持つ。

![alt text](images/2-1.png)

<table>
    <caption>TCP/IP　階層モデル</caption>
    <tbody>
        <tr>
            <td></td>
            <th>層</th>
            <th>説明</th>
            <th>プロトコル</th>
        </tr>
        <tr>
            <th>5</th>
            <td>アプリケーション層</td>
            <td>データの送信や取り扱い、通信手順や方法を実現する役割を持つ。<br><font color=red>この層の多くはC/Sモデルで実装される</font>。</td>
            <td>HTTP, HTML, SMTP, FTP, SNMP, TELENT, SSH</td>
        </tr>
        <tr>
            <th>4</th>
            <td>トランスポート層</td>
            <td>アプリケーションプログラム間の通信を実現する役割を持つ。<br><font color=red>プログラム間の通信はポート番号により識別され、<br>複数のプログラムが同時に動作している</font>。</td>
            <td>TCP, UDP</td>
        </tr>
        <tr>
            <th>3</th>
            <td>インターネット層</td>
            <td>ネットワークの細かい構造を抽象化し、<br>IPを使用してIPアドレスをもとにパケットを転送する。</td>
            <td>IP, ICMP, ARP, ...</td>
        </tr>
        <tr>
            <th>2</th>
            <td>ネットワーク<br>インタフェース層</td>
            <td>イーサネットなどのデータリンクを利用して通信するための<br>インタフェースとなる階層であり、デバイスドライバ(※2)に相当する。<br></td>
            <td>デバイスドライバの<br>層のため特になし。</td>
        </tr>
        <tr>
            <th>1</th>
            <td>ハードウェア(※1)</td>
            <td>物理的にデータを転送してくれるイーサネットや電話回線などを指す。<br>通信媒体(ケーブル、無線)や信頼性やセキュリティ、帯域、遅延時間などの<br>内容については何も決めていない。</td>
            <td>ハードウェアの<br>層のため特になし。</td>
        </tr>
    </tbody>
</table>

※1. 第一層と第二層をまとめて1つの層として扱うこともある。
※2. OSとハードウェアの橋渡しをするソフトウェア。
※3. RDP(リモートデスクトップ)はRFCには規定されていない。

#### インターネット層のプロトコル

```plantuml
@startuml
title インターネット層のイメージ

node "ホストA" as hostA1
node "ホストB" as hostB1
rectangle インターネット as internet1 {
    rectangle "機器" as machine1
    rectangle "機器" as machine2
    rectangle "機器" as machine3
}
node "ホストA" as hostA2
node "ホストB" as hostB2
cloud インターネット as internet2

hostA1 -> machine1 : イーサネット
machine1 -> machine2 : ATM
machine2 -> machine3 : 専用回線
machine3 -> hostB1 : イーサネット
hostA2 -> internet2
internet2 -> hostB2
internet1 =[#red]=> internet2 : <color red>ネットワークの細かい構造を抽象化
hostA1 -[hidden]-- hostA2
hostB1 -[hidden]--hostB2

@enduml
```

- **IP(Internet Protocol)**: ネットワークを跨いでインターネット全体にパケットを送り届けるためのプロトコル。<font color=red>IPにはデータリンクの特性を隠す役割</font>もあり、通信経路がどのようなデータリンクになっていても通信可能になっている。
- **ICMP(Internet Control Message Protocol)**: IPパケット配送中に異常が発生した場合、送信元に異常を知らせるプロトコル。ネットワーク診断などに利用される。
- **ARP(Address Resolution Protocol)**: IPアドレスからMACアドレス(パケットの送り先)を取得するプロトコル。逆に、MACアドレスからIPアドレスを取得するプロトコルをRARP(Reverse ARP)と呼ぶ。

#### トランスポート層のプロトコル

```plantuml
@startuml
title トランスポート層のイメージ\n(論理的な通信路)
left to right direction

node クライアント as client {
    rectangle Webブラウザ as web_browser
    rectangle メールソフト as mail_soft
    rectangle "遠隔ログイン\nクライアント" as remote_login_client
}
node サーバ as server {
    rectangle Webサーバ as web_server
    rectangle メールサーバ as mail_server
    rectangle "遠隔ログイン\nサーバ" as remote_login_server
}
client =u= server : 物理的な通信路
web_browser -- web_server : ポート番号X
mail_soft -- mail_server : ポート番号Y
remote_login_client -- remote_login_server : ポート番号Z

@enduml
```

| 比較項目             | TCP (Transmission Control Protocol)     | UDP (User Datagram Protocol)  |
|----------------------|------------------------------------------------------|-----------------------------------------------------------|
| 接続の確立           | <font color=red>データ転送前に通信相手との接続を確立する必要がある。</font>       | <font color=red>接続の確立は不要。データを直接送信できる。</font>                    |
| 信頼性               | 高い。パケットの順序確認、再送制御、エラー検出を行うが、遅延が発生する可能性がある   | 低い。パケット送信だけであり遅延はないが、順序保証や再送制御はなく、エラー検出も限定的 |
| データ転送の順序     | パケットの順序を保証する                                | 順序は保証されない                                         |
| 再送制御             | パケットが欠落した場合、自動的に再送される               | 再送制御は行われない                                       |
| 使用例           | 信頼性が重要なアプリケーション<br>(例：ファイル転送、メール) | リアルタイム性が重要なアプリケーション<br>(例：VoIP、ゲーム、ビデオや音声などのマルチメディア通信)    |

#### アプリケーション層(セッション層以上の上位置)のプロトコル

```plantuml
@startuml
title アプリケーション層のイメージ\n(クライアント/サーバモデル)
left to right direction

node クライアント as client {
    rectangle Webブラウザ as web_browser
    rectangle メールソフト as mail_soft
    rectangle "遠隔ログイン\nクライアント" as remote_login_client
}
node サーバ as server {
    rectangle Webサーバ as web_server
    rectangle メールサーバ as mail_server
    rectangle "遠隔ログイン\nサーバ" as remote_login_server
}

server --> client : 応答
server <-up- client : 要求
web_browser -[hidden]- web_server
mail_soft -[hidden]- mail_server
remote_login_client -[hidden]- remote_login_server

@enduml
```

##### 【ユースケース】 Webアクセス(WWW)

```plantuml
@startuml
title WWW(World Wide Web)のイメージ
left to right direction

node "Webクライアント\n・PC\n・スマホ\n・タブレット" as client
node WebサーバA as serverA
node WebサーバB as serverB
node WebサーバC as serverC
cloud "\n\nインターネット\n\n" as internet
note bottom of client
・HTTPがアプリケーション層のプロトコル
・HTMLがプレゼンテーション層のプロトコル
end note

client -[#red]-> internet : <color red>①要求
client <-[#red]- internet
client -[#blue]-> internet : <color blue>①要求
client <-[#blue]- internet
client -[#green]-> internet : <color green>①要求
client <-[#green]- internet
internet -[#red]-> serverA
internet -[#blue]-> serverB
internet -[#green]-> serverC
serverA -[#red]-> internet : <color red>②応答
serverB -[#blue]-> internet : <color blue>②応答
serverC -[#green]-> internet : <color green>②応答

@enduml
```

- WWW(World Wide Web)はインターネットが一般に普及する原動力になったアプリケーション

###### 【ユースケース】 電子メール(E-Mail)

```plantuml
@startuml
title 電子メールのイメージ

node ホストA as hostA
node ホストB as hostB
queue 伝送路 as line {
    file 電子メール as mail
}
note bottom of mail
MIMEがプレゼンテーション層のプロトコル
end note

hostA -> mail
mail -> hostB

@enduml
```

- SMTP(Simple Message Transfer Protocol)では、テキストや映像、音声などを送信できる。
- 送信データは<b>MIME(Multipurpose Internet Mail Extensions)</b>という仕様に基づいている。

###### 【ユースケース】 ファイル転送

```plantuml
@startuml
title 電子メールのイメージ
left to right direction

actor Aさん as UserA
node ホストA as hostA {
    folder フォルダA as folderA {
        file ファイルA as fileA
    }
}
node ホストB as hostB {
    folder フォルダB as folderB {
        file ファイルA as fileB
    }
}

UserA -- hostA : 利用
fileA ..> fileB : "      ファイル転送       "

@enduml
```

- FTPによるファイル転送はバイナリモードやテキストモードを選択可能であり、これが**プレゼンテーション層の機能**ということができる。
- FTPでは、<font color=red><b>FTPの制御用ポート(21番)</b>と<b>データ転送用ポート(20番)</b>の2つ</font>があり、これら2つのTCPコネクションを制御すること**がセッション層の機能**と言える。

##### 【ユースケース】 遠隔ログイン

```plantuml
@startuml
title 遠隔ログイン(TELNETとSSH)のイメージ

actor Aさん as UserA
node ホストA as hostA
node ホストB as hostB
note top of hostB
ホストAの前にいるAさんが
ネットワーク経由でホストBに遠隔ログイン。
end note

UserA -> hostA : ログイン
hostA => hostB : 遠隔ログイン

@enduml
```

- 遠隔ログインのプロトコルとしてTELNETやSSHが用意されている。
- <font color=red>リモートデスクトップ接続のプロトコルである<b>RDPはTCP/IPのRFCで規定されているプロトコルではない</b></font>。

##### 【ユースケース】 ネットワーク管理

```plantuml
@startuml
title "SNMP(Simple Network Management Protocol)のイメージ"
left to right direction

actor Aさん as UserA
node "ネットワーク\n管理端末\n(SNMPマネージャ)" as SnmpManager
note bottom of SnmpManager
・SNMPがアプリケーション層のプロトコル
・MIBがプレゼンテーション層のプロトコル
end note
cloud "\n\nLAN, インターネット\n\n" as internet
node "パソコン\nワークステーション\nサーバなど\n(SNMPエージェント)" as pc
node "ルータ\n(SNMPエージェント)" as router
node "スイッチ\n(SNMPエージェント)" as switch

UserA - SnmpManager
SnmpManager -[#red]-> internet : <color red>①設定変更
SnmpManager <-[#red]- internet
SnmpManager -[#blue]-> internet : <color blue>①状態確認
SnmpManager <-[#blue]- internet
SnmpManager -[#green]-> internet : <color green>①状態確認
SnmpManager <-[#green]- internet
internet -[#red]-> pc
internet <-[#red]- pc : <color red>②NIC情報・通信パケット通知
internet -[#blue]-> router
internet <-[#blue]- router : <color blue>②機器の温度通知
internet -[#green]-> switch
internet <-[#green]- switch : <color green>②設定内容・異常パケット通知

@enduml
```

- SNMP(Simple Network Management Protocol)はネットワーク管理で利用されるプロトコルである。
- ネットワーク管理を行う端末をSNMPマネージャ、状態を管理される端末をSNMPエージェントと呼ぶ。
- SNMPでは、<font color=red>MIB(Management Informtion Base)と呼ばれる決められたデータ構造</font>で情報が格納される。
- SNMPや各種動作ログを活用し、将来のネットワーク拡張のために情報収集する。