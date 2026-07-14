## 複数のテーブルの関係を表現するER図

<div style="padding: 10px; margin-bottom: 10px; border: 5px double;">
    <h5>学習のポイント</h5>
    <ul>
        <li>テーブル(エンティティ)数の増加によるテーブル同士の複雑さを軽減・解消するためにER図を使用する。</li>
        <li>ER図の描き方には何種類かフォーマットがあり、IEとIDEF1Xの表記法がある。</li>
        <li>RDBにおけるテーブル間の関係は「1対多」が原則。「多対多」の場合は関連実体を用いる。</li>
    </ul>
</div>

<div style="padding: 10px; margin-bottom: 10px; border: 5px double;">
    <h5>勘どころ</h5>
    <ol start=33>
        <li><p style="display: flex;">IE表記法のカーディナリティの記号は「◯：0」「ー：1」「<img src="images/crow_feet.png"/>(鳥の足)：2以上」の3つ</p></li>
        <li>従属エンティティは主キーに外部キーが含まれ、独立エンティティは主キーに外部キーが含まれていない。</li>
    </ol>
</div>

### IE表記法

```plantuml
!include puml/4_1IE.puml
```

### 「多対多」と関連実体

関連実体は「多対多」の関係にあるエンティティの主キーをそれぞれ組み合わせたエンティティであり、**RDB側の都合によって導入される人工的なエンティティ**である。「多対多」の状態を解消するための必須の技術になる。

```plantuml
!include puml/4_2NonRelationEntity.puml
```
```plantuml
!include puml/4_3RelationEntity.puml
```

<!-- 改ページ -->
<div style="page-break-before:always"></div>

### 演習問題

#### 問4-1： 以下の五つのテーブルを、IE表記法を用いてER図を記述せよ。

<!-- 支社テーブル、支店テーブル、商品分類テーブル -->
<table>
    <tr>
        <td>
            <table>
                <caption>支社</caption>
                <thead>
                <tr>
                    <th>支社コード</th>
                    <th>支社名</th>
                </tr>
                </thead>
                <tr>
                    <td>001</td>
                    <td>東京</td>
                </tr>
                <tr>
                    <td>002</td>
                    <td>大阪</td>
                </tr>
            </table>
        </td>
        <td>
            <table>
                <caption>支店</caption>
                <thead>
                <tr>
                    <th>支社コード</th>
                    <th>支店コード</th>
                    <th>支店名</th>
                </tr>
                </thead>
                <tr>
                    <td>001</td>
                    <td>01</td>
                    <td>渋谷</td>
                </tr>
                <tr>
                    <td>001</td>
                    <td>02</td>
                    <td>八重洲</td>
                </tr>
                <tr>
                    <td>002</td>
                    <td>01</td>
                    <td>堺</td>
                </tr>
                <tr>
                    <td>002</td>
                    <td>02</td>
                    <td>豊中</td>
                </tr>
            </table>
        </td>
        <td>
            <table>
                <caption>商品分類</caption>
                <thead>
                <tr>
                    <th>商品分類コード</th>
                    <th>分類名</th>
                </tr>
                </thead>
                <tr>
                    <td>C1</td>
                    <td>水洗用品</td>
                </tr>
                <tr>
                    <td>C2</td>
                    <td>食器</td>
                </tr>
                <tr>
                    <td>C3</td>
                    <td>書籍</td>
                </tr>
                <tr>
                    <td>C4</td>
                    <td>日用雑貨</td>
                </tr>
            </table>
        </td>
    </tr>
</table>

<!-- 支店商品テーブル、商品テーブル -->
<table>
    <tr>
        <td>
            <table>
                <caption>支店商品</caption>
                <thead>
                <tr>
                    <th>支社コード</th>
                    <th>支店コード</th>
                    <th>商品コード</th>
                </tr>
                </thead>
                <tr>
                    <td>001</td>
                    <td>01</td>
                    <td>001</td>
                </tr>
                <tr>
                    <td>001</td>
                    <td>01</td>
                    <td>002</td>
                </tr>
                <tr>
                    <td>001</td>
                    <td>01</td>
                    <td>003</td>
                </tr>
                <tr>
                    <td>001</td>
                    <td>02</td>
                    <td>002</td>
                </tr>
                <tr>
                    <td>001</td>
                    <td>02</td>
                    <td>003</td>
                </tr>
                <tr>
                    <td>001</td>
                    <td>02</td>
                    <td>004</td>
                </tr>
                <tr>
                    <td>001</td>
                    <td>02</td>
                    <td>005</td>
                </tr>
                <tr>
                    <td>001</td>
                    <td>02</td>
                    <td>006</td>
                </tr>
                <tr>
                    <td>002</td>
                    <td>01</td>
                    <td>001</td>
                </tr>
                <tr>
                    <td>002</td>
                    <td>01</td>
                    <td>002</td>
                </tr>
                <tr>
                    <td>002</td>
                    <td>02</td>
                    <td>007</td>
                </tr>
                <tr>
                    <td>002</td>
                    <td>02</td>
                    <td>008</td>
                </tr>
            </table>
        </td>
        <td>
            <table>
                <caption>商品</caption>
                <thead>
                <tr>
                    <th>商品コード</th>
                    <th>商品名</th>
                    <th>商品分類コード</th>
                </tr>
                </thead>
                <tr>
                    <td>001</td>
                    <td>石鹸</td>
                    <td>C1</td>
                </tr>
                <tr>
                    <td>002</td>
                    <td>タオル</td>
                    <td>C1</td>
                </tr>
                <tr>
                    <td>003</td>
                    <td>歯ブラシ</td>
                    <td>C1</td>
                </tr>
                <tr>
                    <td>004</td>
                    <td>コップ</td>
                    <td>C1</td>
                </tr>
                <tr>
                    <td>005</td>
                    <td>箸</td>
                    <td>C2</td>
                </tr>
                <tr>
                    <td>006</td>
                    <td>スプーン</td>
                    <td>C2</td>
                </tr>
                <tr>
                    <td>007</td>
                    <td>雑誌</td>
                    <td>C3</td>
                </tr>
                <tr>
                    <td>008</td>
                    <td>爪切り</td>
                    <td>C4</td>
                </tr>
            </table>
        </td>
    </tr>
</table>

##### 回答

```plantuml
!include puml/4_4IE_exer.puml
```

#### 問4-2： 上記5テーブルの中で関連実体を列挙せよ。

##### 回答

支店商品エンティティが関連に該当する。

#### 問4-3： 現実世界に存在する多対多の関連を三つ列挙し、関連実体を考えよ。

##### 回答

- **1.消費者と商品**： 消費者と商品は多対多の関係になる。解消する関連実体は、「購入」または「注文」になる。
- **2.学生と部活**： 学生は一人が複数の部活を掛け持ちでき、一つの部活には複数の学生が所属する。解消する関連実体は「所属」になる。
- **3.著者と書籍**： 著者と書籍は多対多の関係になる。解消する関連実体「著述」になる。

```plantuml
!include puml/4_4RelationEntity_exer.puml
```