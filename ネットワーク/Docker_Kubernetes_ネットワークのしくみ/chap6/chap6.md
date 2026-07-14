<style>
    body {
      counter-reset: chapter 5;
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

# サービスメッシュを支える技術

## サービスメッシュとIstio

### サービスメッシュとは



### Istioとは



### Istioのアーキテクチャ



### サイドカーとサイドカーフリー



### まとめ



## Istioの使い方としくみ

### 環境構築



### サンプルアプリの起動



### クラスタ外部からのアクセス許可



### タイムアウト



### 認証



### Kialiによるアーキテクチャの可視化



### まとめ


