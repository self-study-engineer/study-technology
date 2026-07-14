## 必ずわかるウィンドウ関数

<div style="padding: 10px; margin-bottom: 10px; border: 5px double;">
    <h5>学習のポイント</h5>
    <ul>
        <li>ウィンドウ関数の「ウィンドウ」とは「(順序を持つ)範囲」という意味</li>
        <li><font color=red><b>ウィンドウ関数は①レコード集合のカット(PARTITION BY)、②レコードの順序付け(ORDER BY)、③サブセットの定義(ROWSやRANGE)の3つの機能を持つ。</font></b></li>
        <li>PARTITION BY句はGROUP BY句から集約の機能を引いて、カットの機能だけを残し、ORDER BY句はレコードの順序を付ける。</li>
        <li>フレーム句はカーソルの機能をSQLの構文に持ち込むことで「カレントレコード」を中心にしたレコード集合の範囲を定義することができる。</li>
        <li>ウィンドウ関数の内部動作としては現在のところ、レコードのソートが行われている。将来的にハッシュが採用される可能性もゼロではない。</li>
    </ul>
</div>

ウィンドウ関数は行間比較を容易に行えるようになる。本章では、ウィンドウ関数の基本的理解に焦点を当てて記述する。具体的なクエリについては7章を参照。

<div style="page-break-before:always"></div>

#### ウィンドウとは何か

<table>
  <caption>shohinテーブル</caption>
  <thead>
    <tr>
      <th>商品ID</th>
      <th>商品名</th>
      <th>商品分類</th>
      <th>単価</th>
      <th>原価</th>
      <th>登録日</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>0001</td><td>Tシャツ</td><td>衣服</td><td>1000</td><td>500</td><td>2009-09-20</td></tr>
    <tr><td>0002</td><td>穴あけパンチ</td><td>事務用品</td><td>500</td><td>320</td><td>2009-09-11</td></tr>
    <tr><td>0003</td><td>カッターシャツ</td><td>衣服</td><td>4000</td><td>2800</td><td></td></tr>
    <tr><td>0004</td><td>包丁</td><td>キッチン用品</td><td>3000</td><td>2800</td><td>2009-09-20</td></tr>
    <tr><td>0005</td><td>圧力鍋</td><td>キッチン用品</td><td>6800</td><td>5000</td><td>2009-01-15</td></tr>
    <tr><td>0006</td><td>フォーク</td><td>キッチン用品</td><td>500</td><td></td><td>2009-09-20</td></tr>
    <tr><td>0007</td><td>おろしがね</td><td>キッチン用品</td><td>880</td><td>790</td><td>2008-04-28</td></tr>
    <tr><td>0008</td><td>ボールペン</td><td>事務用品</td><td>100</td><td></td><td>2009-11-11</td></tr>
  </tbody>
</table>

上記テーブルを用いてウィンドウ関数の基本的動作を理解する。例えば以下のウィンドウ関数は<u>商品IDを昇順でソートし、2つ前までの商品を含む価格の移動平均</u>を求めている。どちらも出力結果は同じであり、無名ウィンドウと名前付きウィンドウの違いがある。

```sql
-- 【無名ウィンドウ構文】商品IDを昇順でソートし、2つ前までの商品を含む価格の移動平均を求める
SELECT 
	shohin_id, 
	shohin_mei, 
	hanbai_tanka,
	AVG (hanbai_tanka) OVER (ORDER BY shohin_id
		ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM shohin;

-- 【名前付きウィンドウ構文】商品IDを昇順でソートし、2つ前までの商品を含む価格の移動平均を求める
SELECT shohin_id, shohin_mei, hanbai_tanka,
	AVG (hanbai_tanka) OVER W AS moving_avg
FROM shohin
WINDOW W AS (ORDER BY shohin_id 
	ROWS BETWEEN 2 PRECEDING AND CURRENT ROW);
```

<div style="page-break-before:always"></div>

上記のように無名ウィンドウ構文と名前付きウィンドウ関数はそれぞれ以下の効果がある。

- 【**無名ウィンドウ構文の効果**】簡略的に表現できる。ほとんどのDBMSで使用可能。
- 【**名前付きウィンドウ構文の効果**】使い回しが可能だが、構文エラーになるDBMSも存在する。例えばOracleなど。

名前付きウィンドウ構文は**①共通表式(CTE: Common Table Expression)によるビューの使い回し**や**②名前月プロシージャの定義**と同じ効果がある。例えば、以下のように使いまわすことができる。

```sql
-- 名前付きウィンドウ構文を用いたウィンドウの使い回し
SELECT shohin_id, shohin_mei,
	AVG (hanbai_tanka) OVER W AS moving_avg,
	SUM(hanbai_tanka) OVER W AS moving_sum,
	COUNT(hanbai_tanka) OVER W AS moving_count,
	MAX(hanbai_tanka) OVER W AS moving_max
FROM shohin
WINDOW W AS (ORDER BY shohin_id　ROWS BETWEEN 2 PRECEDING AND CURRENT ROW);
```

#### 1枚でわかるウィンドウ関数

ウィンドウ関数の機能とその俯瞰した図を示す。1と2はそれぞれGROUP BYとORDER BYの機能とほぼ同じであり、3はカーソル(cursor)を取り込んだものである。ただし、PARTITION BYとGROUP BYは同じではなく、`PARTITION BY=GROUP BY - 集約`と覚えておけば良い。詳細は18章を参照。

1. PARTITION BY句によるレコード集合のカット
2. ORDER BY 句によるレコードの順序付け
3. FRAME句によるカレントレコードを中心としたサブセットの定義

<img src="images/ウィンドウ関数のイメージ.png" width=90%>

<div style="page-break-before:always"></div>

#### フレーム句を使って違う行を自分の行に持ってくる

フレーム句はカレントレコードを基準として、「統計指標」や「異なる行を自分の行に持ってくる」など応用場面が多くある。例えば、以下のテーブルから直近の日付を求めるクエリを考える。

<table>
  <caption>loadsample</caption>
  <thead>
    <tr>
      <th>sample_date</th>
      <th>load_val</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>2018-02-01</td><td>1024</td></tr>
    <tr><td>2018-02-02</td><td>2366</td></tr>
    <tr><td>2018-02-05</td><td>2366</td></tr>
    <tr><td>2018-02-07</td><td>985</td></tr>
    <tr><td>2018-02-08</td><td>780</td></tr>
    <tr><td>2018-02-12</td><td>1000</td></tr>
  </tbody>
</table>

```sql
-- 【名前付きWINDOW構文】過去の直近の日付と負荷量を求める
SELECT 
	sample_date AS cur_date,
	load_val AS cur_val,
	MIN(sample_date) OVER W AS latest_date,
	MIN(load_val) OVER W AS latest_val
FROM loadsample
WINDOW W AS (
	ORDER BY sample_date ASC 
	ROWS BETWEEN 1 PRECEDING AND 1 PRECEDING
);

-- 出力結果
-- cur_date	cur_val	latest_date	latest_val
-- 2018-02-01	1024		
-- 2018-02-02	2366	2018-02-01	1024
-- 2018-02-05	2366	2018-02-02	2366
-- 2018-02-07	985	2018-02-05	2366
-- 2018-02-08	780	2018-02-07	985
-- 2018-02-12	1000	2018-02-08	780
```

上記のウィンドウ関数を踏まえ、FAQをまとめる。

##### Q1.フレームは前だけでなく「後ろ」にも移動させることは可能か？

**→回答: 可能。**

FOLLOWINGキーワードを使用することで対応可能。

```sql
SELECT 
	sample_date AS cur_date,
	load_val AS cur_val,
	MIN(sample_date) OVER W AS next_date,
	MIN(load_val) OVER W AS next_val
FROM loadsample
WINDOW W AS (
	ORDER BY sample_date ASC 
	ROWS BETWEEN 1 FOLLOWING AND 1 FOLLOWING -- 1 FOLLOWING = 1行後
);

-- 出力結果
-- cur_date	cur_val	next_date	next_val
-- 2018-02-01	1024	2018-02-02	2366
-- 2018-02-02	2366	2018-02-05	2366
-- 2018-02-05	2366	2018-02-07	985
-- 2018-02-07	985	2018-02-08	780
-- 2018-02-08	780	2018-02-12	1000
-- 2018-02-12	1000		
```

##### Q2.MIN関数を使っているが、これには何か意味があるのか？

**→回答: フレームの範囲を1行にしている場合は意味はないが、複数行の場合は意味がある。**

例えば、Q1の場合は1行を対象にしているため意味がない。2行以上になると意味がある。<u>MIN関数だけでなく、MAXやSUM、AVG関数でも同様のことが言える。</u>

<div style="page-break-before:always"></div>

##### Q3.行ではなく「1日前」や「2日前」のように列の値に基づいたフレームも設定可能か？

**→回答: 可能。**

「ROWS」の代わりに「RANGE」というキーワードを使用する。<font color=red><u>RANGEは列のデータ型を意識する必要があり、数値や日付・時間で利用する場面が多い。</u></font>下記クエリでは日付の間隔を指定している。

```sql
-- 1日前の結果があれば出力するクエリ
SELECT sample_date AS cur_date,　load_val AS cur_val,
	MIN(sample_date) OVER W AS day1_before,
	MIN(load_val) OVER W AS load_day1_before
FROM loadsample
WINDOW W AS ( -- RANGEは「データ型」を意識する
	ORDER BY sample_date ASC 
	RANGE BETWEEN interval '1' day PRECEDING AND interval '1' day PRECEDING
);

-- 出力結果
-- cur_date	cur_val	day1_before	load_day1_before
-- 2018-02-01	1024		
-- 2018-02-02	2366	2018-02-01	1024
-- 2018-02-05	2366		
-- 2018-02-07	985		
-- 2018-02-08	780	2018-02-07	985
-- 2018-02-12	1000		
```

##### フレーム句で利用できるオプションまとめ

<table>
	<tbody>
		<tr>
			<th>オプション</th>
			<th>用途</th>
		</tr>
		<tr>
			<td>ROWS</td>
			<td>移動単位を「行」で設定する</td>
		</tr>
		<tr>
			<td>RANGE</td>
			<td>移動単位を「列の値」で設定する。<br>基準となる列はORDER BY句で指定された列</td>
		</tr>
		<tr>
			<td>n PRECEDING</td>
			<td>nだけ前へ(小さい方)へ移動する。　※nは正の整数</td>
		</tr>
		<tr>
			<td>n FOLLOWING</td>
			<td>nだけ後へ(大きい方)へ移動する。　※nは正の整数</td>
		</tr>
		<tr>
			<td>UNBOUNDED PRECEDING</td>
			<td>無制限に遡る方へ移動する。</td>
		</tr>
		<tr>
			<td>UNBOUNDED FOLLOWING</td>
			<td>無制限に下る方へ移動する。</td>
		</tr>
		<tr>
			<td>CURRENT ROW</td>
			<td>現在行</td>
		</tr>
	</tbody>
</table>

<div style="page-break-before:always"></div>

##### ウィンドウ関数まとめ

```sql
SELECT shohin_mei, shohin_bunrui, hanbai_tanka,
	ROW_NUMBER()      OVER () AS row_number,
	RANK()            OVER (ORDER BY hanbai_tanka) AS rank,
	DENSE_RANK()      OVER (ORDER BY hanbai_tanka) AS dense_rank,
	MAX(hanbai_tanka) OVER () AS max_tanka,
	MIN(hanbai_tanka) OVER () AS min_tanka,
	SUM(hanbai_tanka) OVER () AS sum_tanka,
	COUNT(*)          OVER (PARTITION BY shohin_bunrui) AS shohin_count
FROM shohin;
```

<table>
  <caption>出力結果</caption>
  <thead>
    <tr>
      <th>shohin_mei</th>
      <th>shohin_bunrui</th>
      <th>hanbai_tanka</th>
      <th>row_number</th>
      <th>rank</th>
      <th>dense_rank</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>ボールペン</td><td>事務用品</td><td>100</td><td>1</td><td>1</td><td>1</td></tr>
    <tr><td>穴あけパンチ</td><td>事務用品</td><td>500</td><td>2</td><td>2</td><td>2</td></tr>
    <tr><td>おろしがね</td><td>キッチン用品</td><td>880</td><td>3</td><td>3</td><td>3</td></tr>
    <tr><td>Tシャツ</td><td>衣服</td><td>1000</td><td>4</td><td>4</td><td>4</td></tr>
    <tr><td>包丁</td><td>キッチン用品</td><td>3000</td><td>5</td><td>5</td><td>5</td></tr>
    <tr><td>カッターシャツ</td><td>衣服</td><td>4000</td><td>6</td><td>6</td><td>6</td></tr>
    <tr><td>圧力鍋</td><td>キッチン用品</td><td>6800</td><td>7</td><td>7</td><td>7</td></tr>
</tbody>
</table>

<table>
  <caption>出力結果(続き)</caption>
  <thead>
    <tr>
      <th>max_tanka</th>
      <th>min_tanka</th>
      <th>sum_tanka</th>
      <th>shohin_count</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>6800</td><td>100</td><td>16280</td><td>2</td></tr>
    <tr><td>6800</td><td>100</td><td>16280</td><td>2</td></tr>
    <tr><td>6800</td><td>100</td><td>16280</td><td>3</td></tr>
    <tr><td>6800</td><td>100</td><td>16280</td><td>2</td></tr>
    <tr><td>6800</td><td>100</td><td>16280</td><td>3</td></tr>
    <tr><td>6800</td><td>100</td><td>16280</td><td>2</td></tr>
    <tr><td>6800</td><td>100</td><td>16280</td><td>3</td></tr>
  </tbody>
</table>



<div style="page-break-before:always"></div>

#### 演習問題

<table>
  <caption>ServerLoadSample</caption>
  <thead>
    <tr>
      <th><u>server</th>
      <th><u>sample_date</th>
      <th>load_val</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>A</td><td>2018-02-01</td><td>1024</td></tr>
    <tr><td>A</td><td>2018-02-02</td><td>2366</td></tr>
    <tr><td>A</td><td>2018-02-05</td><td>2366</td></tr>
    <tr><td>A</td><td>2018-02-07</td><td>985</td></tr>
    <tr><td>A</td><td>2018-02-08</td><td>780</td></tr>
    <tr><td>A</td><td>2018-02-12</td><td>1000</td></tr>
    <tr><td>B</td><td>2018-02-01</td><td>54</td></tr>
    <tr><td>B</td><td>2018-02-02</td><td>39008</td></tr>
    <tr><td>B</td><td>2018-02-03</td><td>2900</td></tr>
    <tr><td>B</td><td>2018-02-04</td><td>556</td></tr>
    <tr><td>B</td><td>2018-02-05</td><td>12600</td></tr>
    <tr><td>B</td><td>2018-02-06</td><td>7309</td></tr>
    <tr><td>C</td><td>2018-02-01</td><td>1000</td></tr>
    <tr><td>C</td><td>2018-02-07</td><td>2000</td></tr>
    <tr><td>C</td><td>2018-02-16</td><td>500</td></tr>
  </tbody>
</table>

上記テーブルをもとに、問題2-1と2-2を回答せよ。

<div style="page-break-before:always"></div>

##### 問題2-1 ウィンドウ関数の結果よそう その1

下記SQLを実行した際の実行結果を予想せよ。

```sql
SELECT 
    server,
    sample_date,
    SUM(load_val) OVER () AS sum_load
FROM ServerLoadSample;
```

##### 問題2-1の回答

まず、実行結果は以下の通りである。クエリにはPARTITION BY句がないため、全行の合計を出力するようになっている。

```sql
-- 出力結果
server	sample_date	sum_load
A   	2018-02-01	74448
A   	2018-02-02	74448
A   	2018-02-05	74448
A   	2018-02-07	74448
A   	2018-02-08	74448
A   	2018-02-12	74448
B   	2018-02-01	74448
B   	2018-02-02	74448
B   	2018-02-03	74448
B   	2018-02-04	74448
B   	2018-02-05	74448
B   	2018-02-06	74448
C   	2018-02-01	74448
C   	2018-02-07	74448
C   	2018-02-16	74448
```

<div style="page-break-before:always"></div>

##### 問題2-2 ウィンドウ関数の結果よそう その2

下記SQLを実行した際の実行結果を予想せよ。

```sql
SELECT 
    server,
    sample_date,
    SUM(load_val) OVER (PARTITION BY server) AS sum_load
FROM ServerLoadSample;
```

##### 問題2-2の回答

まず、実行結果は以下の通りである。クエリにはPARTITION BY句によりserver単位で区切られており、serverごとの合計が算出されている。serverがAであれば8521、Bであれば62427、Cであれば3500になっている。

```sql
-- 出力結果
server	sample_date	sum_load
A   	2018-02-01	8521
A   	2018-02-02	8521
A   	2018-02-05	8521
A   	2018-02-07	8521
A   	2018-02-08	8521
A   	2018-02-12	8521
B   	2018-02-01	62427
B   	2018-02-02	62427
B   	2018-02-03	62427
B   	2018-02-04	62427
B   	2018-02-05	62427
B   	2018-02-06	62427
C   	2018-02-01	3500
C   	2018-02-07	3500
C   	2018-02-16	3500
```
