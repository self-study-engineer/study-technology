<style>
    body {
      counter-reset: chapter 0;
    }
    h1 {
        counter-reset: sub-chapter;
    }
    h2 {
        counter-reset: section;
    }

    h1::before {
        counter-increment: chapter;
        content: "第" counter(chapter) "章 ";
    }
    h2::before {
        counter-increment: sub-chapter;
        content: counter(chapter) "-" counter(sub-chapter) " ";
    }
</style>

# イントロダクション

## 本書の目的と対象読者

- 近年Webシステムの規模が拡大し、`Dokcer`や`Kebernetes`のようなコンテナ技術を活用してクラウド上でシステム構築する方法が採用されつつある。本書では、`Dokcer`と`Kubernetes`の動きを理解する上で必要となるネットワーク技術に絞って解説する。
- **対象読者**は①`Docker`、`Kubernetes`を使ってアプリ開発を行なっている方、②`Docker`、`Kubernetes`を使ってクラスタシステムやマイクロサービスを構築/運用している方、③クラウドネイティブを支えるネットワーク技術を学びたい方、としている。

## 本書の読み方

- **【2章】Dockerネットワークの要素技術**
<u>本書で取り扱う`Dokcer`と`Kubernetes`のネットワーク技術について理解しやすくするため</u>に、Dockerのネットワークを理解する上での要素技術を順番に説明する。ネットワークの基本概念やプロトコル、IPアドレスなど、ネットワーク技術に関する基本知識を身につける。
- **【3章】Dockerネットワークのしくみ**
<u>`Docker`を用いた通信や構成要素のアーキテクチャを理解し、効率的なシステム構築やトラブルシューティングを可能にするため</u>に、`Docker`の①コンテナ・コンテナ間、②ホストマシン・コンテナ間、③マルチホストネットワーク間の3種類の通信とネットワーク構成について解説する。
- **【4章】Kubernetesネットワークにしくみ**
<u>`Kubernetes`のネットワーク関連の全体像と裏側のしくみを理解し、より効果的な運用や開発を可能にするため</u>に、ネットワーク全体の構成やネットワーク関連リソースのしくみ、コンポーネントの動作について説明する。
- **【5章】CNI(Container Network Interface)**
<u>大規模なコンテナ環境でも効率的なネットワーク管理を可能にするため</u>に、`Kubernetes`との連携がスムーズに行える`CNI`の全体像やプラグイン、`Kubernetes`リソースとの組み合わせについて解説する。`CNI`はさまざまなコンテナランタイムやネットワークプラグインを自由に組み合わせることができ、独自のネットワーク要件に対応することが容易になる。
- **【6章】サービスメッシュを支える技術**
<u>マイクロサービスのネットワーク管理や通信の最適化に役立つ知識や技術を身につけるため</u>に、サービスメッシュの基本概念や`Istio`の基本機能、実践的な使用方法について説明する。サービスメッシュの利用により、運用管理や監視、セキュリティなどの面で効率的な対応が可能になり、開発や運用の負担軽減が期待できる。

## 使用する環境とソフトウェア

<table>
  <thead>
    <tr>
      <th>環境</th>
      <th>マシン</th>
      <th>OS</th>
      <th>アーキテクチャ</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>AWS環境</td>
      <td>AWS(Amazon Web Services)<br>EC2 インスタンス</td>
      <td>Ubuntu 24.04 LTS</td>
      <td>x86</td>
    </tr>
    <tr>
      <td>PC環境</td>
      <td>MacBook Pro</td>
      <td>macOS Sequoia 15.3.1</td>
      <td>ARM64<br>(Apple M2チップ)</td>
    </tr>
  </tbody>
</table>

- 【**ソフトウェアバージョン**】
  - Docker: 27.5.1
  - Python: 3.12.3
  - tcpdump: 4.99.4/4.99.1
  - Kubernetes: 1.32.0
  - minikube: 1.35.0
  - Istio: 1.24.3

### AWSの環境準備

- 【**アプリケーションおよびOSイメージ**】Amazonマシンイメージ(AMI)として「**Ubuntu Server 24.04 LTS(HVM), SSD Volume Type**」を選択、アーキテクチャは「**64ビット(x86)**」を選択。
- 【**インスタンスタイプ**】「**t2.micro**」 ※1vCPU、 1GiBメモリ
- 【**ストレージ設定**】「8GiB」の「gp3」ルートボリュームを選択
- 【**インストールが必要なソフトウェア**】docker(https://docs.docker.com/engine/install/ubuntu/ を参考)、python3(デフォルトインストール)、tcpdump(デフォルトインストール)
- 【**キーペア**】本書専用で環境を用意する場合は新しいキーペアを作成することを推奨する。
- 【**ネットワーク設定**】本書専用で環境を用意する場合は、下表を参考に設定する。

<table>
    <caption><b>ネットワーク設定</caption>
	<tbody>
		<tr>
			<th>用途</th>
			<th>タイプ</th>
			<th>プロトコル</th>
			<th>ポート範囲</th>
			<th>ソースタイプ</th>
			<th>ソース</th>
		</tr>
		<tr>
			<th>sshのため</th>
			<td>ssh</td>
			<td>TCP</td>
			<td>22</td>
			<td>マイIP</td>
			<td>自分のIPアドレスが<br>自動設定される</td>
		</tr>
		<tr>
			<th>ping実行<br>のため</th>
			<td>全ての<br>ICMP-IPv4</td>
			<td>ICMP</td>
			<td>全て</td>
			<td>マイIP</td>
			<td>自分のIPアドレスが<br>自動設定される</td>
		</tr>
		<tr>
			<th>第3章で作る<br>アプリのため</th>
			<td>カスタムTCP</td>
			<td>TCP</td>
			<td>5000</td>
			<td>マイIP</td>
			<td>自分のIPアドレスが<br>自動設定される</td>
		</tr>
	</tbody>
</table>

### PCの環境準備

- 【**インストールが必要なソフトウェア**】docker(Docker Desktop)、python3(デフォルトインストール)、tcpdump(デフォルトインストール)

## 本書のリポジトリとサポートページ

- https://github.com/ShuntaroOkuma/docker-kubernetes-network-book
