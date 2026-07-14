<style>
    body {
      counter-reset: chapter 4;
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
    h3::before {
        counter-increment: section;
        content: counter(chapter) "-" counter(sub-chapter) "-" counter(section) " ";
    }
</style>

# CNI(Container Network Interface)

## CNIのしくみ

### Kubernetesにおけるネットワークの要件



### CNIとは



### 設定ファイル



### 環境変数



### まとめ



## CNIプラグインの裏側のしくみ

### CNIプラグインを支えるバックエンド技術



### 代表的なCNIプラグイン



### Calicoを使ったネットワーク環境の構築



### まとめ



## Network Policy

### Network Policyとは



### podSelector



### namespaceSelector



### Selectorの組み合わせ



### Network Policyのしくみ



### まとめ


