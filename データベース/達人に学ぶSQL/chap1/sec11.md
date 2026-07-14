## SQLを早くする

<div style="padding: 10px; margin-bottom: 10px; border: 5px double;">
    <h5>学習のポイント</h5>
    <ul>
        <li>SQLではユーザが明示的にソートの演算をDBMSに命令することはなく、<font color=red><b>そういった「手続き」を極力ユーザから隠蔽する</b>ことがSQLの設計思想である</font></li>
        <li>INにサブクエリを取る場合はEXISTSまたは結合に書き換える</li>
        <li>インデックスを加工・計算はせず裸で利用する。</li>
        <li>SQLは明示的にソートを記述することはないが、暗黙のソートを行う演算が多くあるので注意が必要</li>
        <li>余計な中間テーブルはなるべく減らし、低速ストレージへのアクセスを減らす。</li>
        <li>レコード数を絞れる条件は早い段階(JOINやWHERE)で記述する。負債ははやく返さないとあとでツケを払うことになる。</li>
    </ul>
</div>

##### SQLの実行順序

```sql
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

##### ソート発生タイミング

- `GROUP BY`句
- `ORDER BY`句
- 集約関数(`SUM, COUNT, AVG, MAX, MIN`)
- `DISTINCT`
- 集合演算子(`UNION, INTERSECT, EXCEPT`)
- ウィンドウ関数(`RANK, DENSE_RANK, ROW_NUMBER`等)

<div style="page-break-before:always"></div>

#### 効率の良い検索を利用する

<table>
    <tr>
        <td>
            <table>
                <caption>Class_A</caption>
                <thead>
                    <tr>
                        <th><u>id</th>
                        <th>name</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>1</td><td>田中</td></tr>
                    <tr><td>2</td><td>鈴木</td></tr>
                    <tr><td>3</td><td>伊集院</td></tr>
                </tbody>
            </table>
        </td>
        <td>
            <table>
                <caption>Class_B</caption>
                <thead>
                    <tr>
                        <th><u>id</th>
                        <th>name</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>1</td><td>田中</td></tr>
                    <tr><td>2</td><td>鈴木</td></tr>
                    <tr><td>4</td><td>西園寺</td></tr>
                </tbody>
            </table>
        </td>
    </tr>
</table>

##### サブクエリを引数に取る場合、INよりもEXISTSを使う

上記のテーブルを用いて`IN`と`EXISTS`のSQLを考える。一般的に`IN`より`EXISTS`の方が処理が速い。理由は以下の通り。

- 結合キーにインデックスが貼られていれば<u>インデックスを参照するのみで済む</u>から。
- `EXISTS`は<u>条件に合致する行を見つけた時点で検索を打ち切る</u>から。

```sql
-- IN述語(処理速度×、可読性○)
SELECT * FROM class_a
WHERE id IN (SELECT id FROM class_b);

-- EXISTS句(処理速度○、可読性×)
SELECT * FROM class_a A
WHERE EXISTS (SELECT id FROM class_b B　WHERE A.id = B.id);

-- 出力結果
id	name
1	田中
2	鈴木
```

##### サブクエリを引数に取る場合、INよりも結合を使う

<font color=red>INのパフォーマンス改善には`EXISTS`だけでなく、結合に書き換える方法も知られている</font>。フラット化により、少なくともどちらかのテーブルのインデックスが利用でき、パフォーマンス向上が期待できる。インデックスがない場合は`EXISTS`の方が軍配が上がる。

```sql
-- INを結合で代用
SELECT * FROM class_a A INNER JOIN class_b B ON A.id = B.id
```

<div style="page-break-before:always"></div>

#### ソートを回避する

SQLでは、**ユーザからソート演算などの「手続き」を極力隠蔽する設計思想を持つ**。しかし、それはDBMS内部でもソートが行われていないということではなく、ソート演算はコストがかかる処理であるため、ソートが発生する演算についてユーザは意識する必要がある。
ソートが発生する代表的な演算は以下の通り。

- `GROUP BY`句
- `ORDER BY`句
- 集約関数(`SUM, COUNT, AVG, MAX, MIN`)
- `DISTINCT`
- 集合演算子(`UNION, INTERSECT, EXCEPT`)
- ウィンドウ関数(`RANK, DENSE_RANK, ROW_NUMBER`等)

ソートがメモリ上で処理されている間は問題ないが、メモリ不足によりストレジでソートされるようになると<font color=red>パフォーマンスが大きく低下する。</font>大雑把であるが、<u>メモリとハードディスク(HDD)は数十万〜百万倍の性能差がある</u>。

##### 集合演算子のALLオプションをうまく使う

SQLは`UNION`、`INTERESECT`、`EXCEPT`という3つの集合演算子を持っており、ALLオプションなしで使うと、**必ず重複排除のためのソートが実行される**。

```sql
SELECT * FROM class_a
UNION　-- UNIONは「重複排除する」
SELECT * FROM class_b;

-- 出力結果
id	name
4	西園寺
3	伊集院
1	田中
2	鈴木
```

「重複を気にしなくて良い場合」や「重複が発生しない場合」は`UNION`の代わりに`UNION ALL`を使うことで**ソートによるコストを軽減**することができる。`INTERSECT`と`EXCEPT`でも同様のことが言え、パフォーマンスチューニングに有効である。

```sql
SELECT * FROM class_a
UNION ALL　-- UNION ALLは「重複排除しない」
SELECT * FROM class_b;

-- 出力結果
id	name
1	田中
2	鈴木
3	伊集院
1	田中
2	鈴木
4	西園寺
```

##### DISTINCTをEXISTS句で代用する

`DISTINCT`も重複を排除するためのソートを行うため、処理速度に影響を与える。以下の2つのテーブルを使って説明する。

<table>
    <tr>
        <td>
            <table>
                <caption>Itemsテーブル</caption>
                <thead>
                    <tr>
                        <th><u>item_no</th>
                        <th>item</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>10</td><td>FD</td></tr>
                    <tr><td>20</td><td>CD-R</td></tr>
                    <tr><td>30</td><td>MO</td></tr>
                    <tr><td>40</td><td>DVD</td></tr>
                </tbody>
            </table>
        </td>
        <td>
            <table>
            <caption>SalesHistoryテーブル</caption>
                <thead>
                    <tr>
                        <th><u>sale_date</th>
                        <th><u>item_no</th>
                        <th>quantity</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>2018-10-01</td><td>10</td><td>4</td></tr>
                    <tr><td>2018-10-01</td><td>20</td><td>10</td></tr>
                    <tr><td>2018-10-01</td><td>30</td><td>3</td></tr>
                    <tr><td>2018-10-03</td><td>10</td><td>32</td></tr>
                    <tr><td>2018-10-03</td><td>30</td><td>12</td></tr>
                    <tr><td>2018-10-04</td><td>20</td><td>22</td></tr>
                    <tr><td>2018-10-04</td><td>30</td><td>7</td></tr>
                </tbody>
            </table>
        </td>
    </tr>
</table>

例えば、「売上のあった商品を探すクエリ」として3パターンを考える。「その1」のクエリは重複があり、「その2」のクエリは重複をなくし、「その3」のクエリは最適解である。「その3」の<font color=red>`EXISTS`を用いた方法であればソートは発生せず、比較的高速に動作する</font>。

```sql
-- 売上のあった商品を探すクエリ その1
SELECT I.item_no FROM items I
INNER JOIN saleshistory SH ON I.item_no = SH.item_no;

-- 出力結果
item_no
10
20
30
10 -- 重複している
30 -- 重複している
20 -- 重複している
30 -- 重複している

-- 売上のあった商品を探すクエリ その2
SELECT DISTINCT I.item_no FROM items I
INNER JOIN saleshistory SH ON I.item_no = SH.item_no;

-- 出力結果
item_no
10
20
30

-- 売上のあった商品を探すクエリ その3(最適解)
SELECT I.item_no FROM items I
WHERE EXISTS(
    SELECT SH.item_no FROM saleshistory SH
    WHERE I.item_no = SH.item_no
);

-- 出力結果
item_no
10
20
30
```

<div style="page-break-before:always"></div>

#### 極値関数(MAX/MIN)でインデックスを使う

SQLは`MAX`と`MIN`という2つの極値関数を持っており、両者はいずれもソートを発生させるが、引数の列にインデックスが存在する場合、インデックススキャンにより実表への検索を回避できる。

```sql
-- これは全表検索が必要
SELECT MAX(item) FROM items;

-- これはインデックスを利用できる(検索高速化による軽減措置)
SELECT MAX(item_no) FROM items;
```

#### WHERE句でかける条件はHAVINGには書かない

以下の2つのクエリは出力結果は同じであるが、パフォーマンスの面において違いがある。

- 【**違い1**】`GROUP BY`句による集約はソートやハッシュの演算を行うため、事前に(`WHERE`句で)絞り込んだ方が負荷軽減が期待できる。
- 【**違い2**】`WHERE`句の条件でインデックスを利用でき、絞り込みを効率的に行える。

また、`GROUP BY`句や`ORDER BY`句でインデックスのキーを指定する場合はソート検索を高速化することが可能。特にユニークインデックスを持つ列を指定した場合には、ソート自体をスキップできる実装もある。

```sql
-- 集約した後にHAVING句でフィルタリング
SELECT sale_date, SUM(quantity) FROM saleshistory
GROUP BY sale_date
HAVING sale_date = '2018-10-01';

-- 集約する前にWHERE句でフィルタリング
SELECT sale_date, SUM(quantity) FROM saleshistory
WHERE sale_date = '2018-10-01'
GROUP BY sale_date;

-- 出力結果(上記2つとも同じ結果)
sale_date	sum
2018-10-01	17
```

##### 【補足】SQLの実行順序

```sql
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

<div style="page-break-before:always"></div>

#### そのインデックス本当に使われてますか

##### 【ダメな例1】索引列に加工を行っている

そもそも、SQLは計算向きの言語ではない。そのため、どうしても計算処理をしたい場合は「**索引列(インデックス)は裸のまま**」で加工せずに扱うことで効率的に処理できるようにする。

```sql
-- ×
SELECT * FROM SomeTable
WHERE col_1 * 1.1 > 100; -- 「索引列と掛け算の結果」で検索

-- ○
SELECT * FROM SomeTable
WHERE col_1 > 100 / 1.1; -- 「索引列のみ」で検索
```

##### 【ダメな例2】インデックス列にNULLが存在する

インデックスにおける`NULL`の扱いは困難であり、`NULL`の統一的な扱いの基準がないことから`NULL`は検索条件では使わないことが理想である。そのため、<font color=red>インデックス列の検索条件では、数値であれば最小値、文字列であれば空文字など、`NULL`以外の何かしらの基準値を用意することで対応する必要がある</font>。**ただし**、混乱を招くようなコーディングは避けることは前提にある。

```sql
-- ×
SELECT * FROM SomeTable
WHERE col_1 IS NULL -- インデックスが使用されない場合がある

-- ○ (混乱を招かないSQLであることが前提)
SELECT * FROM SomeTable
WHERE col1 > 0 -- 最小値よりも小さい数を指定(NULL以外)
```

##### 【ダメな例3】否定形を使っている

パフォーマンスを考慮する際、<font color=red>否定形(`<>, !=, NOT IN`)はインデックスを使用できないことは理解しておく必要がある</font>。

```sql
-- ×
SELECT * FROM SomeTable 
WHERE col_1 <> 100; -- 否定形はインデックスを使用できない。
```

##### 【ダメな例4】ORを使っている

`col_1`と`col_2`に別々の索引がある場合、または`(col_1, col_2)`に複合索引を貼っている場合のいずれも`OR`を使って条件を結合すると**①インデックスが利用できなくなる**か、使えたとしても**②`AND`に比べれば非効率的な検索になる**。

```sql
-- ×
SELECT * FROM SomeTable 
WHERE col_1 > 100 OR col_2 = 'abc'; -- OR演算はAND演算よりも非効率
```

##### 【ダメな例5】複合索引の場合に列の順番を間違えている

`(col_1, col_2, col_3)`に対して順番に複合インデックスが貼られているとする。この場合、以下の確認をする必要がある。

- 必ず`col_1`を先頭に書く必要がある。
- `col_1 → col_2 → col_3`の順番を崩してはならない。

もし、上記事項を守れない場合は**別々のインデックスに分割すること**を検討する。

```sql
-- ○
SELECT * FROM SomeTable
WHERE col_1 = 10 AND col_2 = 100 AND col_3 = 500; -- 順番と一致する。

-- ○
SELECT * FROM SomeTable
WHERE col_1 = 10 AND col_2 = 100; -- 順番と一致する。

-- ×
SELECT * FROM SomeTable
WHERE col_1 = 10 AND col_3 = 500; -- 順番と一致しない。

-- ×
SELECT * FROM SomeTable
WHERE col_2 = 100 AND col_3 = 500; -- 順番と一致しない。
```

<div style="page-break-before:always"></div>

##### 【ダメな例6】後方一致または中間一致のLIKE述語を用いている

`LIKE`述語を使うときは、前方一致検索のみ索引が使用される。

```sql
-- × 後方一致はダメ
SELECT * FROM SomeTable 
WHERE col_1 LIKE '%a';

-- × 中間一致はダメ
SELECT * FROM SomeTable 
WHERE col_1 LIKE '%a%';

-- ○ 前方一致のみインデックスが利用される
SELECT * FROM SomeTable 
WHERE col_1 LIKE 'a%';
```

##### 【ダメな例7】暗黙の型変換を行なっている

例えば、文字列型で定義された`col_1`に対する条件を書く場合を考える。暗黙の型変換は以下の悪影響がある。

- オーバヘッドが発生する
- インデックスが使用されない
- エラーになるDBMSもあり、エラー発生の可能性を生む
※PostgreSQLではデータ型が左辺と右辺で異なる場合はエラーになる

```sql
-- × 数値型と比較(暗黙の型変換)
SELECT * FROM SomeTable 
WHERE col_1 = 10;

-- ○ 文字列型と比較
SELECT * FROM SomeTable 
WHERE col_1 = '10';

-- ○ 文字列型でキャスト
SELECT * FROM SomeTable 
WHERE col_1 = CAST(10, AS CHAR(2));
```

<div style="page-break-before:always"></div>

#### 中間テーブルを減らせ

SQLではサブクエリの結果を中間テーブルとして、SQLプログラミングの高い柔軟性を担保いるが、<u>中間テーブルを不用意に使用すればパフォーマンス低下の原因になる</u>。中間テーブルの問題点は以下の通り。

- データ展開に伴うメモリ消費(場合によってはストレージも消費する)
- 中間テーブルで集約した場合、元テーブルに存在したインデックスの使用が困難になる

##### HAVING句を活用しよう

集約結果に対する条件はHAVING句を使って設定するのが原則である。以下にクエリ例を示す。HAVING句は集約を行いながら並行して動作するため、<u>①中間テーブルの作成後に実行されるWHERE句よりも効率的</u>で、<u>②コードを簡潔にまとめられる</u>という利点もある。

```sql
-- × 中間テーブルを使ったクエリ例
SELECT * 
FROM (
	SELECT sale_date, MAX(quantity) AS max_qty
	FROM saleshistory
	GROUP BY sale_date
) TMP -- 無駄な中間テーブル
WHERE max_qty >= 10;

-- ○ HAVING句を用いて中間テーブルの作成を回避したクエリ例
SELECT sale_date, MAX(quantity) AS max_qty
FROM saleshistory
GROUP BY sale_date
HAVING MAX(quantity) >= 10;

-- 出力結果
sale_date	max_qty
2018-10-01	10
2018-10-04	22
2018-10-03	32
```

<div style="page-break-before:always"></div>

##### IN述語で複数のキーを利用する場合は一箇所にまとめる

`SQL-92`から行比較の機能が取り入れられ、`=, <, >`といった比較述語や`IN`の引数にスカラ値ではなく値のリストを取ることが可能になった。具体的を以下に示す。この方法には<u><b>①結合時の型変換を気にしなくて良いこと</u></b>、<u><b>②列に加工を施さなくても良いこと</u></b>の2つの効果がある。

```sql
-- × 2つのサブクエリを使った検索
SELECT family_id, name, address FROM Addresses A1
WHERE name IN (
    SELECT name FROM Addresses2 A2 
    WHERE A1.family_id = A2.family_id
) AND address IN(
    SELECT address FROM Addresses2 A2 
    WHERE A1.family_id = A2.family_id
);

-- ○ ロジックを一箇所にまとめた検索
SELECT family_id, name, address FROM Addresses A1
WHERE family_id || name || address IN (
    SELECT family_id || name || address FROM Addresses A2 
    WHERE A1.family_id = A2.family_id
);

-- ○ 列のペアをINの引数にとる検索
SELECT family_id, name, address FROM Addresses A1
WHERE (family_id, name, address) IN (
    SELECT family_id, name, address FROM Addresses A2 
    WHERE A1.family_id = A2.family_id
);
```

<div style="page-break-before:always"></div>

##### 集約(GROUP BY)よりも結合(JOIN)を先に行う

「結合」と「集約」を併用するケースにおいて、極力、集約よりも結合を行うことで中間テーブルを省略することができる。

##### ビューのご利用は計画的に

ビューは複雑な定義をするとパフォーマンス面で大きなマイナスになる。具体的には<b>①集約関数(`AVG, COUNT, SUM, MIN, MAX`)</b>や<b>②集合演算子(`UNION, INTERSECT, EXCEPT`)</b>を含んだクエリである。これらの欠点を補うためにマテリアライズドビューの技術も実装されているが、メリット・デメリットがあるため、用法・用量には気をつける必要がある。

<table>
    <caption>ビューとその関連用語</caption>
    <thead>
        <tr>
            <th></th>
            <th>データ鮮度</th>
            <th>パフォーマンス</th>
            <th>データを保持</th>
            <th>格納領域</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th>ビュー</th>
            <td>高<br>(リアルタイム)</td>
            <td>低</td>
            <td>しない</td>
            <td>ー</td>
        </tr>
        <tr>
            <th>マテリアライズド<br>ビュー</th>
            <td>中<br>(更新時期次第)</td>
            <td>高</td>
            <td>永続的に保持</td>
            <td>任意(通常は<br>データファイル)</td>
        </tr>
        <tr>
            <th>中間テーブル</th>
            <td>中<br>(更新時期次第)</td>
            <td>中(物理設計と<br>統計情報に注意)</td>
            <td>一時的に保持</td>
            <td>一時ファイル</td>
        </tr>
    </tbody>
</table>

