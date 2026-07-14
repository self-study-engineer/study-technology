###　IPv6のヘッダフォーマット

- <font color=red>ルータの処理軽減のためにIPv6ではチェックサムは省略され、転送速度向上が実現している。また、64ビットCPUのコンピュータで処理しやすい構造にするためにIPv6のヘッダやオプションは全て8オクテット(64ビット)単位で構成される。</font>
- IPv6ヘッダは固定長である。
- 分割処理のための識別子などはオプションになった。

#### IPv6ヘッダ

- **バージョン(4ビット)**: IPのバージョン番号。IPv6は6になる。
- **トラフィッククラス(8ビット)**: IPv4のTOS(DSCP＋ECN)の部分にあたるフィールド。TOSはv4では利用実績がほとんどなかったためv6では削除される予定でしたが、DiffServとECNを使う形で復活した。
- **フローラベル(20ビット)**: 品質制御(QoS: Quality of Service)に利用される想定で定義されたフィールド。具体的なサービス内容は今後の課題になっている。フローラベルを利用しない場合は全て0で埋める。
<font color=red>※フローラベルは品質制御情報を高速に検索するために利用される索引(インデックス)であり、値に特別な意味はなく乱数で決定される。</font>
- **ペイロード長(16ビット)**: ヘッダを除いたデータ部分の長さ。
- **次のヘッダ(8ビット)**: 次のヘッダのプロトコル情報を示すフィールド。IPv6では、TCPやUDPだけでなく、拡張ヘッダのプロトコル番号入る。
- **ホップリミット(8ビット)**: 通過できるルータ数を示すフィールドで、IPv4のTTLと同じ意味。ルータを通過するたびに1つずつ減り、0になったらパケットが破棄される。
- **送信元/宛先IPアドレス(128ビット)**: 送信元/宛先のIPアドレスを表す。
- **IPv6拡張ヘッダ**: IPv6のヘッダとTCPやUDPヘッダの間に挿入されるフィールド。<font color=red>IPパケットを分割する場合には拡張ヘッダを使うことになる。</font>

<img src="images/4-10.png" width=80%>

<div style="page-break-before:always"></div>

<table>
    <caption>IPヘッダのバージョン番号</caption>
    <tr>
        <th>バージョン</th>
        <th>総称</th>
        <th>プロトコル名</th>
    </tr>
    <tr>
        <td>4</td>
        <td>IP</td>
        <td>Internet Protocol</td>
    </tr>
    <tr>
        <td>5</td>
        <td>ST</td>
        <td>ST Datagram Mode</td>
    </tr>
    <tr>
        <td>6</td>
        <td>IPv6</td>
        <td>Internet Protocol version 6</td>
    </tr>
    <tr>
        <td>7</td>
        <td>TP/IX</td>
        <td>TP/IX: The Next Internet</td>
    </tr>
    <tr>
        <td>8</td>
        <td>The P Internet Protocol</td>
        <td>PIP</td>
    </tr>
    <tr>
        <td>9</td>
        <td>TUBA</td>
        <td>TUBA</td>
    </tr>
</table>

#### IPv6拡張ヘッダ

![alt text](images/4-11.png)

- <font color=red>拡張ヘッダはIPv6ヘッダと上位層のヘッダ情報(TCPやUDPなど)の間に挿入される。</font>
- 拡張ヘッダとそのプロトコル番号いくつか種類がある。
  - IPSecを使うときは50(ペイロードの暗号化)と51(認証ヘッダ)が使われる。
  - Mobile IPv6を使うときは60(宛先オプション)と135(モビリティヘッダ)が使われる。

<table>
    <caption>IPv6拡張ヘッダとプロトコル番号</caption>
    <tr>
        <th>拡張ヘッダ</th>
        <th>プロトコル番号</th>
    </tr>
    <tr>
        <td>ホップバイホップオプション(HOPOPT)</td>
        <td>0</td>
    </tr>
    <tr>
        <td>ルーティングヘッダ(IPv6-Route)</td>
        <td>43</td>
    </tr>
    <tr>
        <td>フラグメントヘッダ(IPv6-Frag)</td>
        <td>44</td>
    </tr>
    <tr>
        <td>ペイロードの暗号化(ESP: Encapsulating Security Payload)</td>
        <td>50</td>
    </tr>
    <tr>
        <td>認証ヘッダ(AH: Authentication Header)</td>
        <td>51</td>
    </tr>
    <tr>
        <td>ヘッダの終わり(IPv6-NoNxt)</td>
        <td>59</td>
    </tr>
        <tr>
        <td>宛先オプション(IPv6-Opts)</td>
        <td>60</td>
    </tr>
        <tr>
        <td>モビリティヘッダ(Mobility Header)</td>
        <td>135</td>
    </tr>
</table>