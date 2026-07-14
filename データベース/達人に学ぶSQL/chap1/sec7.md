## ウィンドウ関数で行間比較を行う

<div style="padding: 10px; margin-bottom: 10px; border: 5px double;">
    <h5>学習のポイント</h5>
    <ul>
        <li>SQLで行同士の比較を行うとき、以前は比較対象のテーブルを追加して相関サブクエリを行っていたが、パフォーマンスと可読性が悪く、不評であった。</li>
        <li>ウィンドウ関数は相関サブクエリの問題である可読性を解決し、今後パフォーマンスの改善も見込まれている。</li>
        <li><font color=red>行間比較の代表的な業務要件として「時系列データの経年分析」がある</font></li>
    </ul>
</div>

SQLでは**同じ行内の列同士の比較は容易**である。具体的にはWHERE句を用いて「col1=col2」のように書けば良いだけである。しかし、**異なる行の列同士の比較(行間比較)は簡単ではない**。具体的には<u>相関サブクエリを使うことで対応できるが、可読性が低く、パフォーマンス低下にもつながる</u>。しかしウィンドウ関数は可読性に優れ、相関サブクエリに置き換えられる存在になった。

<div style="page-break-before:always"></div>

#### 成長・後退・現状維持

<table>
  <caption>salesテーブル</caption>
  <thead>
    <tr>
      <th><u>year</th>
      <th>sale</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>1990</td><td>50</td></tr>
    <tr><td>1991</td><td>51</td></tr>
    <tr><td>1992</td><td>52</td></tr>
    <tr><td>1993</td><td>52</td></tr>
    <tr><td>1994</td><td>50</td></tr>
    <tr><td>1995</td><td>50</td></tr>
    <tr><td>1996</td><td>49</td></tr>
    <tr><td>1997</td><td>55</td></tr>
  </tbody>
</table>

上記テーブルを用いて相関サブクエリとウィンドウ関数の違いを示す。

```sql
-- 前年と年商が同じ年度を求める その1 相関サブクエリの利用
SELECT year, sale FROM sales S1
WHERE sale = (SELECT sale FROM sales S2 WHERE S2.year = S1.year - 1)
ORDER BY year;

-- 前年と年商が同じ年度を求める その2 ウィンドウ関数の利用
SELECT year, current_sale 
FROM (SELECT year, 
    sale AS current_sale,
    SUM(sale) OVER ( -- SUM関数を用いているがMINやMAXでも良い
        ORDER BY year RANGE BETWEEN 1 PRECEDING AND 1 PRECEDING
    ) AS pre_sale
FROM sales) TMP 
WHERE pre_sale = current_sale
ORDER BY year;

-- 出力結果
year	sale
1993	52
1995	50
```

<div style="page-break-before:always"></div>

上記のクエリを応用し、前年に比べて成長・後退・現状維持のどれに該当するのかを求めるクエリを考える。

```sql
-- 成長・後退・現状維持を一度に求める その1 相関サブクエリの利用
SELECT year, current_sale,
    CASE 
        WHEN current_sale = pre_sale THEN '→'
        WHEN current_sale > pre_sale THEN '↑'
        WHEN current_sale < pre_sale THEN '↓'
        ELSE '-'
    END AS var
FROM (SELECT year, sale AS current_sale,
    ( -- 相関サブクエリ
        SELECT sale FROM sales S2　WHERE S2.year = S1.year - 1
    ) AS pre_sale
FROM sales S1) TMP ORDER BY year;

-- 成長・後退・現状維持を一度に求める その2 ウィンドウ関数の利用
SELECT year, current_sale,
    CASE 
        WHEN current_sale = pre_sale THEN '→'
        WHEN current_sale > pre_sale THEN '↑'
        WHEN current_sale < pre_sale THEN '↓'
        ELSE '-'
    END AS var
FROM (SELECT year, sale AS current_sale,
    SUM(sale) OVER ( -- ウィンドウ関数
        ORDER BY year RANGE BETWEEN 1 PRECEDING AND 1 PRECEDING
    ) AS pre_sale
FROM sales) TMP ORDER BY year;

-- 出力結果
year	current_sale    var
1990	50              -
1991	51              ↑
1992	52              ↑
1993	52              →
1994	50              ↓
1995	50              →
1996	49              ↓
1997	55              ↑
```

<div style="page-break-before:always"></div>

#### 時系列に歯抜けがある場合〜直近と比較〜

<table>
    <caption>sales2</caption>
    <thead>
        <tr>
            <th><u>year</th>
            <th>sale</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>1990</td><td>50</td></tr>
        <tr><td>1992</td><td>50</td></tr>
        <tr><td>1993</td><td>52</td></tr>
        <tr><td>1994</td><td>55</td></tr>
        <tr><td>1997</td><td>55</td></tr>
    </tbody>
</table>

上記テーブルを用いて「直近の年と同じ年商の年」の抽出を考える。上記テーブルは1991年、1995年、1996年が歯抜けになっている。

```sql
-- 直近の年と同じ年商の年を選択する その1 相関サブクエリ
SELECT year, sale FROM sales2 S1
WHERE S1.sale = ( -- 直近の年と同じ年商
    SELECT sale FROM sales2 S2
    WHERE S2.year = (
        SELECT MAX(year) FROM sales2 S3 
        WHERE S1.year > S3.year
    )
) ORDER BY year;

-- 直近の年と同じ年商の年を選択する その1 ウィンドウ関数
SELECT year, current_sale FROM (
    SELECT year, sale AS current_sale,
        MAX(sale) OVER (
            ORDER BY year ROWS BETWEEN 1 PRECEDING AND 1 PRECEDING) AS pre_sale
    FROM sales2
)
WHERE current_sale = pre_sale -- 直近の年と同じ年商
ORDER BY year;

-- 出力結果
year	current_sale
1992	50
1997	55
```

<div style="page-break-before:always"></div>

#### ウィンドウ関数 vs. 相関サブクエリ

まず、相関サブクエリとウィンドウ関数は以下の違いがある。

- ウィンドウ関数はサブクエリであるが、「相関サブクエリ」ではないため、ウィンドウ関数単体で実行可能。
- <font color=red><b>ウィンドウ関数は可読性が高く</b></font>、一度のテーブルスキャンで済むため、<font color=red><b>パフォーマンスが良い</b></font>。
- ウィンドウ関数はレコードを集約せずそのまま元のテーブルに列として結果を追加するだけという**情報保全性が働く利点**がある。

##### なぜウィンドウ関数で相関サブクエリを置き換えられるのか

<table>
  <caption>shohinテーブル</caption>
  <thead>
    <tr>
      <th>shohin_id</th>
      <th>shohin_mei</th>
      <th>shohin_bunrui</th>
      <th>hanbai_tanka</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>0001</td><td>Tシャツ</td><td>衣服</td><td>1000</td></tr>
    <tr><td>0002</td><td>穴あけパンチ</td><td>事務用品</td><td>500</td></tr>
    <tr><td>0003</td><td>カッターシャツ</td><td>衣服</td><td>4000</td></tr>
    <tr><td>0004</td><td>包丁</td><td>キッチン用品</td><td>3000</td></tr>
    <tr><td>0005</td><td>圧力鍋</td><td>キッチン用品</td><td>6800</td></tr>
    <tr><td>0006</td><td>フォーク</td><td>キッチン用品</td><td>500</td></tr>
    <tr><td>0007</td><td>おろしがね</td><td>キッチン用品</td><td>880</td></tr>
    <tr><td>0008</td><td>ボールペン</td><td>事務用品</td><td>100</td></tr>
  </tbody>
</table>

上記テーブルから「平均単価より高い商品」を選択し、相関サブクエリとウィンドウ関数の置き換え可能であることを示す。ウィンドウ関数は相関サブクエリより可読性とパフォーマンス共に優れるため、使わない理由がない。また、<u>レコードの集約もないことから情報保全性が高く、ウィンドウ関数を用いた列を追加するだけで済む</u>。

```sql
-- 平均単価より高い商品 その1 相関サブクエリ
SELECT shohin_bunrui, shohin_mei, hanbai_tanka FROM shohin S1
WHERE hanbai_tanka > (
    SELECT AVG(hanbai_tanka) FROM shohin S2
    WHERE S1.shohin_bunrui = S2.shohin_bunrui
    GROUP BY shohin_bunrui
);

-- 平均単価より高い商品 その2 ウィンドウ関数
SELECT shohin_bunrui, shohin_mei, hanbai_tanka FROM (
    SELECT　*, AVG(hanbai_tanka) OVER (PARTITION BY shohin_bunrui) AS avg_tanka
    FROM shohin
) TMP
WHERE hanbai_tanka > avg_tanka;

-- 出力結果
shohin_bunrui   shohin_mei      hanbai_tanka
事務用品    	穴あけパンチ	500
衣服        	カッターシャツ	4000
キッチン用品	包丁        	3000
キッチン用品	圧力鍋      	6800
```

さらにウィンドウ関数を用いたページネーションの例を示す。

```sql
-- ページネーションの例1
SELECT * FROM (
    SELECT *,
        ROW_NUMBER() OVER (ORDER BY col_1) as row_num
    FROM table_name
) arranged_table
WHERE 1 <= row_num AND row_num <= 10; -- offset(1) <= row_num <= limit(10)

-- ページネーションの例2
SELECT * FROM (
    SELECT
        user_id, registration_date, score,
        ROW_NUMBER() OVER (ORDER BY registration_date ASC) AS row_num
    FROM table_name
) TMP
WHERE row_num BETWEEN (page - 1) * 20 + 1 AND page * 20 -- 1ページあたり20個表示
ORDER BY score DESC;
```

<div style="page-break-before:always"></div>

#### オーバーラップする期間を調べる

<table>
    <caption>reservations(2人間でオーバーラップがある)</caption>
    <thead>
        <tr>
            <th><u>reserver</th>
            <th>start_date</th>
            <th>end_date</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>木村</td><td>2018-10-26</td><td>2018-10-27</td></tr>
        <tr><td>荒木</td><td>2018-10-28</td><td>2018-10-31</td></tr>
        <tr><td>堀</td><td>2018-10-31</td><td>2018-11-01</td></tr>
        <tr><td>山本</td><td>2018-11-03</td><td>2018-11-04</td></tr>
        <tr><td>内田</td><td>2018-11-03</td><td>2018-11-05</td></tr>
        <tr><td>水谷</td><td>2018-11-06</td><td>2018-11-06</td></tr>
    </tbody>
</table>

上記テーブルをガントチャートで書くと以下の通り。

 <img src="images/ガントチャート.png" width=60%>

 期間重複のパターンは以下の3種類であるが、3.については1.と2.を共に満たしていることと同値である。

1. 開始日が他の期間内にある
2. 終了日が他の期間内にある
3. 開始日と終了日ともに他の期間内にある(1.と2.を共に満たす)

上記テーブルからオーバーラップする期間を求めるクエリを以下に示す。
　まず相関サブクエリについては`EXISTS`を用いて表現している。<font color=red>`NOT EXISTS`句を使った場合は「どの期間ともオーバーラップしていない期間」を求めるクエリになる。</font>
　次にウィンドウ関数について、こちらは`start_date`でソートし、MAX関数(MIN関数でも良い)を用いて次の客の投宿日が現在行の客の滞在期間と重なっているデータを出力する。

```sql
-- オーバーラップする期間を求める その2 相関サブクエリの利用
SELECT * FROM reservations R1
WHERE EXISTS (
    SELECT * FROM reservations R2
    WHERE R1.reserver <> R2.reserver -- これがないと全員出力される
    AND (
        R1.start_date BETWEEN R2.start_date AND R2.end_date OR
        R1.end_date BETWEEN R2.start_date AND R2.end_date
    )
);

-- 出力結果
reserver	start_date	end_date
荒木	2018-10-28	2018-10-31
堀	2018-10-31	2018-11-01
内田	2018-11-03	2018-11-05
山本	2018-11-03	2018-11-04
```
```sql
-- オーバーラップする期間を求める その2 ウィンドウ関数
SELECT reserver, next_reserver FROM (
    SELECT 
        reserver,
        start_date,
        end_date,
        MAX(reserver) OVER (
            ORDER BY start_date ASC ROWS BETWEEN 1 FOLLOWING AND 1 FOLLOWING
        ) AS next_reserver,
        MAX(start_date) OVER (
            ORDER BY start_date ASC ROWS BETWEEN 1 FOLLOWING AND 1 FOLLOWING
        ) AS next_start_date
    FROM reservations
)
WHERE next_start_date BETWEEN start_date AND end_date;

-- 出力結果
reserver	next_reserver
荒木	堀
内田	山本
```

<div style="page-break-before:always"></div>

さらに、以下のように3人間でオーバーラップがある場合を考える。クエリは上記と同じものを用いる。出力結果を以下に示す。
　<font color=red>相関サブクエリは重複している事実のみが出力され、誰が重複しているかはデータから判断が必要である一方、ウィンドウ関数は誰と重複しているかまでわかる</font>。
　補足として、山本氏の投宿日(`start_date`)を`11-04`に変更した場合、相関サブクエリの出力結果から内田氏が消えるが、ウィンドウ関数では結果は変わらず正しく出力される。このような点からも<u>**ウィンドウ関数は汎用性が高い**</u>ことがわかる。

<table>
    <caption>reservations(<b>3人間</b>でオーバーラップがある)</caption>
    <thead>
        <tr>
            <th><u>reserver</th>
            <th>start_date</th>
            <th>end_date</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>木村</td><td>2018-10-26</td><td>2018-10-27</td></tr>
        <tr><td>荒木</td><td>2018-10-28</td><td>2018-10-31</td></tr>
        <tr><td>堀</td><td>2018-10-31</td><td>2018-11-01</td></tr>
        <tr><td>山本</td><td>2018-11-03</td><td>2018-11-04</td></tr>
        <tr><td>内田</td><td>2018-11-03</td><td>2018-11-05</td></tr>
        <tr><td><b>水谷(start_dateを変更)</td><td>2018-11-04</td><td>2018-11-06</td></tr>
    </tbody>
</table>

```sql
-- 出力結果(相関サブクエリ)
reserver	start_date	end_date
荒木        	2018-10-28	2018-10-31
堀      	2018-10-31	2018-11-01
山本        	2018-11-03	2018-11-04
内田        	2018-11-03	2018-11-05
水谷        	2018-11-04	2018-11-06

-- 出力結果(ウィンドウ関数)
reserver	next_reserver
荒木        	堀
内田        	山本
山本        	水谷
```

<div style="page-break-before:always"></div>

#### 演習問題

##### 問題7-1

<table>
    <caption>Accounts</caption>
    <thead>
        <tr>
            <th>prc_date</th>
            <th>prc_amt</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>2018-10-26</td><td>12000</td></tr>
        <tr><td>2018-10-28</td><td>2500</td></tr>
        <tr><td>2018-10-31</td><td>-15000</td></tr>
        <tr><td>2018-11-03</td><td>34000</td></tr>
        <tr><td>2018-11-04</td><td>-5000</td></tr>
        <tr><td>2018-11-06</td><td>7200</td></tr>
        <tr><td>2018-11-11</td><td>11000</td></tr>
    </tbody>
</table>

上記テーブルにおいて、ウィンドウ関数を用いて移動平均を求めた場合、下記クエリのようになる。これを相関サブクエリを使って求めよ。

```sql
-- カレント行と前2行を含む3行の移動平均
SELECT prc_date, prc_amt,
    AVG(prc_amt) OVER (
        ORDER BY prc_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS prc_amt_avg
FROM accounts;

-- 出力結果
prc_date	prc_amt	prc_amt_avg
2018-10-26	12000	12000
2018-10-28	2500	7250
2018-10-31	-15000	-166.6666666666666667
2018-11-03	34000	7166.6666666666666667
2018-11-04	-5000	4666.6666666666666667
2018-11-06	7200	12066.6666666666666667
2018-11-11	11000	4400
```

##### 問題7-1の回答

方針としては、`A3.prc_date`が始点`A2.prc_date`と終点`A1.prc_date`の間を動くと考える。定数部分(3>=)を変えることで4行でも5行でも好きな幅で集計対象のウィンドウを移動させることができる。

```sql
-- 相関サブクエリで移動平均を求める
SELECT prc_date, A1.prc_amt, 
(
    SELECT AVG(prc_amt) FROM accounts A2
    WHERE A1.prc_date >= A2.prc_date AND 
        3 >= (SELECT COUNT(*) FROM accounts A3
            WHERE A3.prc_date BETWEEN A2.prc_date AND A1.prc_date)
) AS mvg_sum -- 数値部分(3)を4や5にすることでウィンドウ幅を調整できる
FROM accounts A1 ORDER BY prc_date;

-- 【補足】非グループ化した結果
SELECT A1.prc_date AS "A1date(終点)", A2.prc_date AS "A2date(始点)", A2.prc_amt AS amt
FROM accounts A1, accounts A2
WHERE A1.prc_date >= A2.prc_date AND 
    3 >= (SELECT COUNT(*) FROM accounts A3
        WHERE A3.prc_date BETWEEN A2.prc_date AND A1.prc_date)
ORDER BY "A1date(終点)", "A2date(始点)";

-- 【補足】非グループ化のクエリの出力結果
A1date(終点)	A2date(始点)	amt
2018-10-26	2018-10-26	12000   -- レコード数が足りず除外
2018-10-28	2018-10-26	12000   -- レコード数が足りず除外
2018-10-28	2018-10-28	2500    -- レコード数が足りず除外
2018-10-31	2018-10-26	12000   -- S1
2018-10-31	2018-10-28	2500    -- S1
2018-10-31	2018-10-31	-15000  -- S1
2018-11-03	2018-10-28	2500    -- S2
2018-11-03	2018-10-31	-15000  -- S2
2018-11-03	2018-11-03	34000   -- S2
2018-11-04	2018-10-31	-15000  -- S3
2018-11-04	2018-11-03	34000   -- S3
2018-11-04	2018-11-04	-5000   -- S3
2018-11-06	2018-11-03	34000   -- S4
2018-11-06	2018-11-04	-5000   -- S4
2018-11-06	2018-11-06	7200    -- S4
2018-11-11	2018-11-04	-5000   -- S5
2018-11-11	2018-11-06	7200    -- S5
2018-11-11	2018-11-11	11000   -- S5
```

<div style="page-break-before:always"></div>

##### 問題7-2

前問(問題7-1)において、3行に満たない場合はNULLを出力するクエリを考えよ。

##### 問題7-2の回答

ウィンドウ関数は件数(cnt)を取得し、3未満ならNULLという条件分岐をCASE式で設定している。ここでも<u>レベルの異なる情報を同じ行に持ってこられるというウィンドウ関数の特性が生きます</u>。

```sql
-- ウィンドウ関数
SELECT prc_date, prc_amt, 
    CASE WHEN cnt = 3 THEN prc_amt_avg ELSE NULL END
FROM (
    SELECT prc_date, prc_amt,
        COUNT(*) OVER ( -- 要素数を取得
            ORDER BY prc_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS cnt,
        AVG(prc_amt) OVER ( -- 平均を求める
            ORDER BY prc_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS prc_amt_avg
    FROM accounts) TMP;

-- 相関サブクエリ
SELECT prc_date, A1.prc_amt, (
    SELECT AVG(prc_amt) FROM accounts A2
    WHERE A1.prc_date >= A2.prc_date
        AND 3 >= ( -- 数値部分(3)を4や5にすることでウィンドウ幅を調整できる
            SELECT COUNT(*) FROM accounts A3
            WHERE A3.prc_date BETWEEN A2.prc_date AND A1.prc_date
        )
    HAVING COUNT(*) = 3 -- HAVING句を追加
) AS prc_amt_avg
FROM accounts A1 ORDER BY prc_date;

-- 出力結果
prc_date	prc_amt	prc_amt_avg
2018-10-26	12000	NULL
2018-10-28	2500	NULL
2018-10-31	-15000	-166.6666666666666667
2018-11-03	34000	7166.6666666666666667
2018-11-04	-5000	4666.6666666666666667
2018-11-06	7200	12066.6666666666666667
2018-11-11	11000	4400
```
