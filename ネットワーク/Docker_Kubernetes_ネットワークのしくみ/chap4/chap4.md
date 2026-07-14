<style>
    body {
      counter-reset: chapter 3;
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

# Kubernetesネットワークのしくみ

## Kubernetesネットワークの全体像

### Kubernetesとネットワーク



### Kubernetesの特徴



### Kubernetesのコンポーネント



### Kubernetesのネットワーク



### まとめ



## Kubernetesの環境構築

### minikubeを用意



### アプリを用意



### Docker環境を切り替え



### コンテナイメージをビルド



## Podの通信のしくみ

### Pod内の通信



### Pod間の通信



### ノードとPodの通信



### まとめ



## Serviceのしくみ

### 事前準備



### Serviceとは



### ClusterIP



### NodePort



### LoadBalancer



### ルーティングと負荷分散



### 名前解決



### まとめ



## ingressのしくみ

### Ingressとは



### 環境



### Ingressの設定方法



### Ingress Controllerと裏側のしくみ



### まとめ



## リソースを適用するときの各コンポーネントの動き方

### Kubernetesのコンポーネント



### 事前準備



### Deploymentが生成されるまでの流れ



### Serviceが生成されるまでの流れ



### まとめ


