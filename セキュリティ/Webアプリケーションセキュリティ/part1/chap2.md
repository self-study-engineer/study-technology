<div class=chap2>

# ブラウザのセキュリティ

<div style="padding: 15px 15px 0px 15px; border: 2px solid; border-radius: 5px;">
この章では以下の項目について説明します。
<ol>
    <li>Webブラウザがユーザーを保護する方法</li>
    <li>Webアプリケーションがリソースを読み込む場所を制限するためのHTTPレスポンスヘッダーの設定方法</li>
    <li>ブラウザによるネットワークおよびディスクアクセスの管理方法</li>
    <li>ブラウザによるクッキーの保護方法</li>
    <li>ブラウザが履歴情報を誤って漏洩してしまう可能性</li>
</ol>
</div>

## ブラウザの構成要素

- 

<div style="page-break-before: always;"></div>

## JavaScript サンドボックス

<img src="images/2-1.png" width=85%>

- ブラウザではWebページのHTMLの`<script>`タグによって読み込まれたJavaScriptコードは実行のためにJavaScriptエンジンに渡されます。
- JavaScriptは通常、Webページを動的にするために使用され、ユーザーがページとやり取りするのを待ち、それに応じてページの一部を更新します。
  - <font color=red>`<script>`タグに`defer`属性がある場合、ブラウザはDOMが確定するまでJavaScriptの実行を待ちます</font>が、そうでない場合は即座に実行されます。
  - **インラインJavaScript**の場合はHTMLに直接記述されており、**外部読み込み**の場合は`src`属性で外部URLを指定します。
- JavaScriptエンジンはコードに多くの制限を設けており、これらは**サンドボックス化**と呼ばれる。これにより、ホストシステムに大きなダメージを与えることなく実行できる環境を確保します。<u>具体的にはサンドボックス環境では以下の操作ができません</u>。
  - ディスク上の任意のファイルアクセス
  - 他のOSのプロセスへの干渉や通信
  - OSのメモリの任意の場所を読み取る
  - 任意のネットワークコール
- 【<b>ヒント</b>】ブラウザ上で実行されるJavaScriptの使用目的は一般的にかなり限定的であるため、このトピックでは<u>最初の重要なセキュリティ対策として、Webアプリケーション上のJavaScriptを可能な限り制限することを推奨します</u>。ハッカーはXSS、CSRF、SSRF、XSSIなどの攻撃により被害を引き起こすためリスク軽減のために推奨されます。
- Webページ上のJavaScriptをロックダウンするにはいくつか主要な方法があります。その対策のために次の3つのチェックを行い、安全性を確保します。

> ### JavaScriptコードに対する3つのチェック
> 1. このページで実行できるJavaScriptコードは何か？
> → 【**回答**】コンテンツセキュリティポリシー（**CSP**）を設定する
> 2. このページのJavaScriptにどのようなタスクを実行させるべきか
> → 【**回答**】**CSP**を設定し、**同一オリジンポリシー**を制限する。異なるオリジンからの読み込みはクロスオリジンリソース共有（**CORS**）を設定する。
> <font color=red>※ほとんどのシナリオではCORSヘッダーは何も設定しないことが最も安全なオプション</font>
> 3. 正しいJavaScriptコードを実行しているとどのように確認できるか？
> → 【**回答**】サブリソース完全性（**SRI**：SubResource Integrity）を確保するために、`<script>`タグに`integrity`属性を付与し、実行されるJavaScriptファイル（コード）に対してハッシュ値を求める。

### コンテンツセキュリティポリシー

- 

【**JavaScript**】

```js
const express = require("express")
const app = express()
const port = 3000
app.get("/", (req, res) => { // レスポンスヘッダーとして直接設定
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
        ...
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

### 同一オリジンポリシー

- 例えば、次のオリジンが`https://www.example.com`と同じオリジンかどうかを考える
  - 【`https://www.example.com/profile`】はい。パスが異なっていますが、<font color=red>オリジン（①プロトコル、②ドメイン、③ポート）</font>が一致しています。
  - 【`http://www.example.com`】いいえ。**プロトコル（http）** が異なります。
  - 【`https://www.example.org`】いいえ。**ドメイン（org）** が異なります。
  - 【`https://www.example.com:8080`】いいえ。**ポート（8080）** が異なります。
  - 【`https://blog.example.com`】いいえ。**サブドメイン（blog）** が異なります。
- 

<div style="page-break-before: always;"></div>

### クロスオリジンリクエスト

- ブラウザでは、他のWebサイトへのリンクをクリックしてブラウザがそのサイトを開く際に発生する**クロスオリジン書き込みは許可されています**。
- クロスオリジン埋め込み（画像インポート）はブラウザのCSPが許可すれば、使用できます。
- **クロスオリジン読み込みは事前にブラウザに明示的に指示しない限り許可されません**。この設定はクロスオリジンリソース共有（CORS）設定と呼ばれ、次の3つが考えられます。
  1. 【**バックエンドコード**】最も一般的な設定箇所。Express、FastAPI、Spring などで使用し、許可するオリジンが複数あるときは許可リストと照合します。`Access-Control-Allow-Credentials: true`と組み合わせると`＊`が使えず、オリジンを明示する必要があります。許可リストを環境変数などで環境ごとに切り替えられます。
  2. 【**Webサーバー／リバースプロキシ（Nginx, Apacheなど）**】静的ファイル（フォント、画像、JSONなど）を直接配信する場合に使います。アプリ側に手を入れられない、または複数のアプリの前段でまとめて制御したい場合にも使います。<font color=red>単一オリジンを固定で返すだけなら簡単ですが複数オリジンの出し分けは map などが必要になり、少し面倒</font>。
  3. 【**クラウドの設定**】サーバーレスやマネージドサービス中心の構成では、ここで設定するのが自然。`S3`の場合、バケットのCORS設定があり、静的ホスティングやブラウザから直接アップロードするときに使います。`API Gateway`の場合、HTTP API には組み込みのCORS設定があります。REST API はプロキシ統合だとLambda側でヘッダーを返す必要があり、ハマりやすいポイントです。`CloudFront`の場合、`Response Headers Policy`で付与します。

#### 色々なCORS設定

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
app.get("/api/data", (req, res) => { res.json({ message: "ok" }); });
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

<div style="page-break-before: always;"></div>

### サブリソースの安全性チェック

- 攻撃者がWebサーバにコマンドラインアクセスし、直接既存のJavaScriptコードを編集することを考えます。例えば、**CDN**（コンテンツ配信ネットワーク）にホストされている場合、攻撃者はそれらのシステムを侵害し、悪意のあるスクリプトを置き換えることができます。
- これらの脅威から保護するためにWebページの`<script>`タグにサブリソースの完全性チェックを使用することができます。
    ```html
    <!-- コード例 -->
    <script 
        src="/js/application.js"
        integrity="sha384-503lno38vOKjoSa8HT863w10M7hKzvj+HjknFmPkOJz50htAHuPtPLj6J6lfziE"
    ></script>
    ```
- 実際にハッシュ値を計算する方法としては次の3つがある。
  - **①コマンドで自分で計算する**：例えば以下の実行コマンドの場合、出力された文字列の先頭に sha384- を付けたものが値になります。
    ```bash
    openssl dgst -sha384 -binary lib.js | openssl base64 -A
    ```
  - **②CDNが公開している値を使う**：cdnjsやjsDelivrなどは、ライブラリごとにintegrity付きのscriptタグを用意しています。
  - **③ビルドツールで自動付与する**：webpackやViteのSRIプラグインを使うと、ビルドのたびに自動で付与されます。ファイルを更新するとハッシュも変わるので、<font color=red><b>実運用ではこの方法が確実</b></font>です。
- 【<b>ヒント</b>】SRIチェックはオプションですが、中間者攻撃（MITM）や悪意ある変種から保護する優れた方法であるため、可能な場合は常に使用するべき。
 

<div style="page-break-before: always;"></div>

## ディスクへのアクセス

- 先ほど、ブラウザ上で実行されるJavaScriptはディスク上の任意の場所にアクセスできないと述べました。しかし実際には、厳密に制御された方法でのみアクセスできます。
- ブラウザがどのようにしてこのアクセスを許可しているのかを見てみましょう。

### ファイルAPI

- ブラウザ上で実行されるJavaScriptがディスクにアクセスする最も明白な方法は**ファイルAPI**を使用することです。`<input type="file">`を用いてファイル選択ダイアログを開いたり、`DataTransfer`オブジェクトを使用してユーザーがファイルをドラッグできる領域を提供したりすることができます。

**【例】コード**

```javascript
// #####################################################
// ##### ファイル入力→入力したファイル情報をコンソール出力 #####
// #####################################################
const fileInput = document.querySelector("input[type=file]") // HTML文書内のファイル入力タグを見つけます。
fileInput.addEventListener("change", () => {
    // ユーザーがアップロードするファイルを選択すると、以下の処理が実行されます。
    const [file] = fileInput.files
    const reader = new FileReader()
    reader.addEventListener("load", () => {
        // コンソールにJavaScriptエンジンがファイルにアクセスできるようになったことを示します。
        console.log(reader.result)
    })
    reader.readAsText(file)
})

// ################################
// ##### ファイルのメタデータ取得 #####
// ################################
const fileInput = document.querySelector("input[type=file]") // HTML文書内のファイル入力タグを見つけます。
fileInput.addEventListener("change", () => {
    const [file] = fileInput.files
    console.log("MIME type: " + file.type)
    console.log("File size: " + file.size)
    console.log("Modified: " + file.lastModifiedDate)
})
```

| プロパティ              | 内容                                       | 例                                               |
| ----------------------- | ------------------------------------------ | ------------------------------------------------ |
| `file.type`             | ファイルの種類を表すMIMEタイプ（文字列）   | `text/plain`、`image/png`、`application/pdf`     |
| `file.size`             | ファイルの大きさ（**バイト**単位の数値）   | `2048`（＝2KB）                                  |
| `file.lastModifiedDate` | ファイルの最終更新日時（Dateオブジェクト） | `Tue Nov 24 2026 23:50:00 GMT+0900 (日本標準時)` |

- 上記例のように、ユーザは意図的にファイルの共有を選択しており、またファイルAPIではファイル自体の操作は許可されていません。この制限により、悪意のあるJavaScriptがディスク上のファイルにウイルスを注入するといったことが防止され、特にJavaScriptコードに対してファイルがどのディレクトリ（フォルダ）から読み込まれたのかを**知らせません**。

### WebStorage

- 

### Indexed DB

- 

<div style="page-break-before: always;"></div>

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

<div style="page-break-before: always;"></div>

## クロスサイトトラッキング

- 

## まとめ

- 

</div>