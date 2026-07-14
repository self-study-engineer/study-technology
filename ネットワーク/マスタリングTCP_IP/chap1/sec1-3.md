###　プロトコルとは

- 様々なプロトコルを体系的にまとめたものを**ネットワークアーキテクチャ**という。
- TCP/IPの他にも、IPX/SPX、AppleTalk、SNA、DECnetなどのネットワークアーキテクチャがある。
- コンピュータ通信開始当初、プロトコルの体系化や標準化は重要ではなく、それぞれのメーカが独自の通信をしていた。
- コンピュータネットワークの導入に伴いマルチベンダでの通信環境が強く望まれるようになる。
- ISO(国際標準化機構)はOSIと呼ばれる通信体系を標準化し、OSI参照モデルがよく参照されるようになった。(OSIで定めるプロトコルは普及していないが。。。)
- プロトコルに準拠すれば、ハードウェアやOSを意識することなく、通信が可能になる。
- <font color=red>TCP/IPはISOの国際標準ではなく、IETF(Internet Engineering Task Force)のプロトコルでありデファクトスタンダードとして広く使用される通信プロトコルである</font>。

| ネットワーク<br>アーキテクチャ | プロトコル | 主な用途 |
| -- | -- | -- |
| TCP/IP | IP, ICMP, TCP, UDP, HTTP,<br>TELNET, SNMP, SMTP | インターネット、LAN |
| IPX/SPX(※1) | IPX, SPX, NPC, ... | パソコン LAN |
| AppleTalk | DDP, RTMP, AEP,<br>ATP, ZIP, ... | 現Apple社製品のLANで<br>使われていた |
| DECnet(※2) | DPR, NSP, SCP, ... | 旧DEC社のミニコンピュータ<br>などで使われていた |
| OSI | FTAM, MOTIS, VT, <br>CMIS/CMIP, CLNP, <br>CONP, ... | ー |
| XNS<br>(Xerox Network Services) | IDP, SPP, PEP, ... | Xerox社ネットワークで<br>主に使われていた |

※1: Novell社が開発販売するNetWareシステムのプロトコル。IPネットワークとは異なる方法で動作する。
※2: 旧DEC社が開発した独自ネットワーク。初期の Peer to Peer ネットワークアーキテクチャの1つ。