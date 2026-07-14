###　無線通信

- <font color=red>IEEE802.11は無線LANプロトコルの物理層とデータリンク層の一部(MAC層)を定義した規格</font>
- IEEE802.11は**CSMA/CA**というアクセス制御方式を採用している。
- 無線LANは媒体共有型(半二重通信)のネットワーク
- 無線LANは盗聴や改ざんを考慮して「**暗号化**」が必要であり、現在はWPA2やその拡張版のWPA3が普及している。また、「**認証**」された接続端末(APやPC)だけが通信できるようなアクセス制御を併用することで、より安全な環境を用意する必要がある。
- 無線LANが使用する電波が他の通信機器と干渉し、動作が不安定になることがある。例えば、電子レンジを使っている近くで2.4GHz帯の無線LANを使用すると通信機能が著しく低下することがある。

#### IEEE802.11

<img src="images/3-4.png" width=50%>

<table>
    <caption>IEEE802.11の比較</caption>
	<tbody>
		<tr>
			<th colspan="2">トランスポート層</th>
			<td colspan="6">TCP/UDP など</td>
		</tr>
		<tr>
			<th colspan="2">ネットワーク層</th>
			<td colspan="6">IP など</td>
		</tr>
		<tr>
			<th rowspan="2">データリンク層</th>
			<th>LLC層</th>
			<td colspan="6">802.2 論理リンク制御</td>
		</tr>
		<tr>
			<th>MAC層</th>
			<td colspan="6">802.11 MAC CSMA/CA</td>
		</tr>
		<tr>
			<th rowspan="4">物理層</th>
			<th>方式</th>
			<td>802.11a</td>
			<td>802.11b</td>
			<td>802.11g</td>
			<td>802.11n</td>
			<td>802.11ac</td>
			<td>802.11ax</td>
		</tr>
		<tr>
			<th>最高速度<br>(理論値)</th>
			<td>最大<br>54Mbps</td>
			<td>最大<br>11Mbps</td>
			<td>最大<br>54Mbps</td>
			<td>最大<br>600Mbps</td>
			<td>【wave1】<br>最大<br>1.3Gbps<br><br>【wave2】<br>最大<br>6.9Gbps</td>
			<td>最大<br>9.6Gbps</td>
		</tr>
		<tr>
			<th>周波数帯</th>
			<td>5GHz</td>
			<td>2.4GHz</td>
			<td>2.4GHz</td>
			<td>2.4GHz/5GHz</td>
			<td>5GHz</td>
			<td>2.4GHz/5GHz</td>
		</tr>
		<tr>
			<th>帯域幅</th>
			<td>20MHz</td>
			<td>26MHz</td>
			<td>20MHz</td>
			<td>20MHz<br>40MHz</td>
			<td>20MHz<br>40MHz<br>80MHz<br>160MHz</td>
			<td>20MHz<br>40MHz<br>80MHz<br>160MHz</td>
		</tr>
	</tbody>
</table>

#### WPA2とWPA3（Wi-Fi Protected Access）

- **WPA2**: Wi-Fi Alliance(無線LANの業界団体)の認証プログラムであるWPAを拡張し、AES(共通鍵暗号)ベースの暗号化プロトコルを採用しており、広く普及している。
- **WPA3**: WPA2を拡張した規格であり、小規模オフィス向けのWPA3-Personalと大規模オフィス向けのWPA3-Enterpriseがある。

#### Bluetooth

- IEEE802.11b/gと同じ2.4GHz帯の通信規格。
- 消費電力は小さく、遮蔽物があっても通信できる（鞄の中でも動くのもそのおかげ）。
- データ伝送速度はバージョン2で3Mbps(実際は2.1Mbps)。
- 電波強度によって1m、10m、100m、400mの通信可能距離を取る。
- 1台のマスタ(親)と1〜7台のスレーブ(子)からなるネットワークを作る。

#### ZigBee

- 家電などに組み込むことを前提に低消費電力・短距離の無線通信を実現する規格
- 最大65536($=2^{16}$)個の端末缶を無線通信で繋ぐ
- 日本で利用可能な2.4GHz帯を利用するものでは最大250kbpsとされている