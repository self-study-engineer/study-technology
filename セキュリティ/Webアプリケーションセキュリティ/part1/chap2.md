<div class=chap2>

# ブラウザのセキュリティ

<div style="padding: 15px 15px 0px 15px; border: 2px solid; border-radius: 5px;">
この章では以下の項目について説明します。
<ol>
    <li></li>
    <li></li>
    <li></li>
    <li></li>
    <li></li>
</ol>
</div>

## ブラウザの構成要素

- 

## JavaScript サンドボックス

> ### JavaScriptコードに対する3つのチェック
> 1. このページで実行できるJavaScriptコードは何か？
> → コンテンツセキュリティポリシー（**CSP**）を設定する
> 2. このページのJavaScriptにどのようなタスクを実行させるべきか
> → **CSP**）を設定し、**同一オリジンポリシー**を制限する。異なるオリジンからの読み込みはクロスオリジンリソース共有（**CORS**））を設定する。
> <font color=red>※ほとんどのシナリオではCORSヘッダーは何も設定しないことが最も安全なオプション</font>
> 3. 正しいJavaScriptコードを実行しているとどのように確認できるか？
> → 

- 

### コンテンツセキュリティポリシー

【**JavaScript**】

```js
const express = require("express")
const app = express()
const port = 3000
app.get("/", (req, res) => {
    // レスポンスヘッダーとして直接設定
    res.set("Content-Security-Policy", "default-src 'self'") // CSP
    res.send("Web app secure!")
})
app.listen(port, () => {
    console.log("Example app listening on port ${port}")
})
```

【**HTML**】

```html
<!doctype html>
<html>
    <head>
        <!-- CSPをmetaタグで設定 -->
        <meta 
            http-equiv="Content-Security-Policy"
            content="default-src 'self'">
        <meta charset="utf-8">
        <title></title>
    </head>
    <body>
        <p>Web app secure!</p>
        <script>
            console.log("I am executing inline!") // インラインJavaScript
        </script>
    </body>
</html>
```

【**CSPの解釈**】

- 【**`defalut-src 'self'; script-src ajax.googleapis.com`**】JavaScriptファイルは`ajax.googleapis.com`から読み込むことができます。その他のすべてのリソースはホストドメインから取得する必要があります。
- 【**`script-src 'self' *.googleapis.com; img-src *`**】JavaScriptファイルはgoogleapis.comまたはそのサブドメインから読み込むことができます。<font color=red>画像はどこからでも読み込むことができます</font>。
- 【**`defalut-src 'https: 'unsafe-inline';`**】すべてのリソースはHTTPS経由で読み込まなければなりません。<font color=red>インラインJavaScript（HTMLファイルの中に直接JavaScriptのコードを書き込むこと）は許可されています</font>。
- 【**`defalut-src https: 'unsafe-eval' 'unsafe-inline'`**】すべてのリソースはHTTPS経由で読み込まなければなりません。インラインJavaScriptは許可されています。`eval(...)`関数を使用して文字列をコードとして評価するJavaScriptも許可されています。
- 

### 同一オリジンポリシー

- 例えば、次のオリジンが`https://www.example.com`と同じオリジンかどうかを考える
  - 【`https://www.example.com/profile`】はい。パスが異なっていますが、<font color=red>①プロトコル、②ドメイン、③ポート</font>が一致しています。
  - 【`http://www.example.com`】いいえ。**プロトコル（http）** が異なります。
  - 【`https://www.example.org`】いいえ。**ドメイン（org）** が異なります。
  - 【`https://www.example.com:8080`】いいえ。**ポート（8080）** が異なります。
  - 【`https://blog.example.com`】いいえ。**サブドメイン（blog）** が異なります。
- 

### クロスオリジンリクエスト

- ブラウザでは、他のWebサイトへのリンクをクリックしてブラウザがそのサイトを開く際に発生する**クロスオリジン書き込みは許可されています**。
- クロスオリジン埋め込み（画像のインポート）はWebサイトのCSPが許可している限り許可されます。
- **クロスオリジン読み込みは事前にブラウザに明示的に指示しない限り許可されません**。この設定はクロスオリジンリソース共有（CORS）設定と呼ばれ、設定箇所としては次の3つが考えられる。
  1. 【**バックエンドコード**】最も一般的な設定箇所。Express、FastAPI、Spring などで使用し、許可するオリジンが複数あるときは許可リストと照合します。`Access-Control-Allow-Credentials: true`（Cookie認証など）と組み合わせると * が使えず、オリジンを明示する必要があります。許可リストを環境変数などで環境ごとに切り替えられます。
  2. 【**Webサーバー／リバースプロキシ（Nginx, Apacheなど）**】静的ファイル（フォント、画像、JSONなど）を直接配信する場合に使います。アプリ側に手を入れられない、または複数のアプリの前段でまとめて制御したい場合にも使います。<font color=red>単一オリジンを固定で返すだけなら簡単ですが複数オリジンの出し分けは map などが必要になり、少し面倒</font>。
  3. 【**クラウドのマネージド設定**】サーバーレスやマネージドサービス中心の構成では、ここで設定するのが自然。`S3`の場合、バケットのCORS設定があり、静的ホスティングやブラウザから直接アップロードするときは、実質これしか選択肢がありません。`API Gateway`の場合、HTTP API には組み込みのCORS設定があります。REST API はプロキシ統合だとLambda側でヘッダーを返す必要があり、ハマりやすいポイントです。`CloudFront`の場合、Response Headers Policy で付与します。オリジンが返したヘッダーを上書きするかどうかの設定もあります。

#### 色々なCORS設定

【**Sprint Boot（Java）**】

```java
@RestController
@RequestMapping("/api/items")
@CrossOrigin(origins = "http://localhost:3000")
public class ItemController {
    @GetMapping
    public List<String> list() {
        return List.of("item1", "item2");
    }
}
```

【**Express（JavaScript）**】

```js
const express = require("express");
const cors = require("cors");
const app = express();

// CORS設定
app.use(cors({
  origin: ["https://app.example.com", "http://localhost:5173"],
  credentials: true,
  methods: ["GET", "POST", "PUT", "DELETE"],
  allowedHeaders: ["Content-Type", "Authorization"]
}));

app.get("/api/data", (req, res) => {
  res.json({ message: "ok" });
});

app.listen(3000);
```

【**FastAPI（Python）**】

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# CORS設定
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://app.example.com", "http://localhost:5173"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/api/data")
def read_data():
    return {"message": "ok"}
```

【**PHP**】

```php
<?php
$allowed_origins = [
    "https://app.example.com",
    "http://localhost:5173",
];

$origin = $_SERVER["HTTP_ORIGIN"] ?? "";
if (in_array($origin, $allowed_origins, true)) {
    header("Access-Control-Allow-Origin: " . $origin);
    header("Access-Control-Allow-Credentials: true");
    header("Vary: Origin");
}

header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS");
header("Access-Control-Allow-Headers: Content-Type, Authorization");

if ($_SERVER["REQUEST_METHOD"] === "OPTIONS") {
    http_response_code(204);
    exit;
}

header("Content-Type: application/json; charset=utf-8");
echo json_encode(["message" => "ok"]);
```

### サブリソースの安全性チェック

- 

## ディスクへのアクセス

- 

### ファイルAPI

- 

### WebStorage

- 

### Indexed DB

- 

## クッキー（Cookie）

- 

### セキュアなクッキー

- 

### HttpOnlyクッキー

- 

### SameSite属性

- 

### 有効期限付きクッキー

- 

### クッキーの無効化

- 

## クロスサイトトラッキング

- 

## まとめ

- 

</div>