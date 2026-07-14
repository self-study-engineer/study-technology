## HAVING句の力

<div style="padding: 10px; margin-bottom: 10px; border: 5px double;">
    <h5>学習のポイント</h5>
    <ul>
        <li><font color=red>全称量化を表現する際、EXISTSはパフォーマンス優位、HAVINGは可読性優位になる。</font></li>
        <li>テーブルは行や順序も持たない。そのためSQLでは原則ソートを記述しない。</li>
        <li>GROUP BY句は過不足ない部分集合を作る。</li>
        <li>WHERE句は集合の要素の性質を調べる道具、HAVING句は集合自身の性質を調べる道具。</li>
        <li><b>RDBなどの集合思考言語はベン図が思考の補助線になる</b></li>
        <li>SQLで検索条件を設定するときは検索対象を見極めることが肝要</li>
        <ul>
            <li>実体1つにつき<font color=red>1行</font>が対応している→要素なので<font color=red>WHERE句</font>を使う。</li>
            <li>実体1つにつき<font color=red>複数行</font>が対応している→集合なので<font color=red>HAVING句</font>を使う。</li>
        </ul>
    </ul>
</div>

<table>
    <caption>集合の性質を調べるための条件の使い方一覧</caption>
	<tbody>
		<tr>
            <th>No.</th>
			<th>条件式</th>
			<th>用途</th>
		</tr>
		<tr>
            <td>1</td>
			<td>COUNT(DISTINCT col) = COUNT(col)</td>
			<td>colの値が一意である</td>
		</tr>
		<tr>
            <td>2</td>
			<td>COUNT(*) = COUNT(col)</td>
			<td>colにNULLが存在しない</td>
		</tr>
		<tr>
            <td>3</td>
			<td>COUNT(*) = MAX(col)</td>
			<td>colは歯抜けのない連番(開始値は1)</td>
		</tr>
		<tr>
            <td>4</td>
			<td>COUNT(*) = MAX(col) - MIN(col) + 1</td>
			<td>colは歯抜けのない連番(開始値は任意の整数)<br>※3の一般化</td>
		</tr>
		<tr>
            <td>5</td>
			<td>MIN(col) = MAX(col)</td>
			<td>colが一つだけの値を持つか、またはNULLである</td>
		</tr>
		<tr>
            <td>6</td>
			<td>MIN(col) * MAX(col) > 0</td>
			<td>全てのcolの符号が同じである</td>
		</tr>
		<tr>
            <td>7</td>
			<td>MIN(col) * MAX(col) < 0</td>
			<td>最大値の符号が正で最小値の符号が負</td>
		</tr>
		<tr>
            <td>8</td>
			<td>MIN(ABS(col)) = 0</td>
			<td>colは少なくとも1つの0を含む</td>
		</tr>
		<tr>
            <td>9</td>
			<td>MIN(col - 定数) = -MAX(col - 定数)</td>
			<td>colの最大値と最小値が<br>指定した定数から同じ幅の距離にある。</td>
		</tr>
	</tbody>
</table>

##### 【補足】No.9について

$$
C(定数)=\frac{col_{max} + col_{min}}{2}
$$

<div style="page-break-before:always"></div>

#### データの歯抜けを探す

<table>
    <caption>SeqTbl</caption>
    <thead>
    <tr>
        <th><u>seq</th>
        <th>name</th>
    </tr>
    </thead>
    <tbody>
        <tr><td>1</td><td>ディック</td></tr>
        <tr><td>2</td><td>アン</td></tr>
        <tr><td>3</td><td>ライル</td></tr>
        <tr><td>5</td><td>カー</td></tr>
        <tr><td>6</td><td>マリー</td></tr>
        <tr><td>8</td><td>ベン</td></tr>
    </tbody>
</table>

上記テーブルを用いて、欠番の有無を調べるクエリと欠番の最小値を返すクエリを考える。実際のクエリとイメージを以下に示す。

<img src="images/全単射.png" width=75%>

```sql
-- 歯抜けの有無を返すクエリ
SELECT '歯抜けあり' AS gap FROM seqtbl
HAVING COUNT(*) <> MAX(seq);

-- 欠番を返すクエリ
SELECT MIN(seq + 1) FROM seqtbl
WHERE seq + 1 NOT IN(SELECT seq FROM seqtbl);
```

上記クエリは「開始値が1」を前提とており、例えば「3, 4, 5, 6, 7」といった連番の場合でも「歯抜けあり」「欠番値8」という結果が得られてしまう。そこで、改善後のクエリを以下に示す。

```sql
-- 【改善版】歯抜けの有無を返すクエリ
SELECT '歯抜けあり' AS gap FROM seqtbl
HAVING COUNT(*) <> MAX(seq) - MIN(seq) + 1; -- 上限値 - 下限値 + 1

-- 【改善版】欠番を返すクエリ
SELECT CASE 
	WHEN COUNT(*) = 0 OR MIN(seq) > 1 THEN 1 -- 下限が1の場合は1を返す
	ELSE ( -- 下限が1の場合は最小の欠番を返す
		SELECT MIN(seq + 1) FROM seqtbl S1
		WHERE NOT EXISTS(SELECT * FROM seqtbl S2 WHERE S1.seq + 1 = S2.seq)
	)
END FROM seqtbl;
```

#### HAVING句でサブクエリ〜最頻値を求める〜

<table>
    <caption>Graduates(卒業生テーブル)</caption>
    <thead>
        <tr>
            <th><u>name</th>
            <th>income</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>サンプソン</td><td>400000</td></tr>
        <tr><td>マイク</td><td>30000</td></tr>
        <tr><td>ホワイト</td><td>20000</td></tr>
        <tr><td>アーノルド</td><td>20000</td></tr>
        <tr><td>スミス</td><td>20000</td></tr>
        <tr><td>ロレンス</td><td>15000</td></tr>
        <tr><td>ハドソン</td><td>15000</td></tr>
        <tr><td>ケント</td><td>10000</td></tr>
        <tr><td>ベッカー</td><td>10000</td></tr>
        <tr><td>スコット</td><td>10000</td></tr>
    </tbody>
</table>

上記テーブルを用いて最頻値を求めるクエリを考える。**①ALL述語を使った方法**と**②極値関数を使った方法**の2つがある。2つとも「〜HAVING COUNT(*) >=」までは全く同じクエリであり、ALLかMAXかの違いがある。
　**ただし**、<u>①は「NULL」に気をつけ、②は「空集合」に気をつける必要がある</u>。

```sql
-- ①ALL述語とを使った方法
SELECT income, COUNT(*) FROM graduates
GROUP BY income
HAVING COUNT(*) >= ALL(
	SELECT COUNT(*) FROM graduates GROUP BY income
);

-- ②極値関数を使った方法
SELECT income, COUNT(*) FROM graduates
GROUP BY income
HAVING COUNT(*) >= (SELECT MAX(cnt) FROM(
	SELECT COUNT(*) AS cnt FROM graduates GROUP BY income
));
```

#### NULLを含まない集合を探す

<table>
	<tbody>
		<tr>
			<td>
                <table>
                    <caption>NullTbl</caption>
                    <thead>
                        <tr>
                            <th>col_1</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr><td>NULL</td></tr>
                        <tr><td>NULL</td></tr>
                        <tr><td>NULL</td></tr>
                    </tbody>
                </table>
            </td>
			<td>
                <table>
                    <caption>Students</caption>
                    <thead>
                        <tr>
                            <th>学生ID</th>
                            <th>学部</th>
                            <th>入学日</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr><td>100</td><td>理学部</td><td>2018-10-10</td></tr>
                        <tr><td>101</td><td>理学部</td><td>2018-09-22</td></tr>
                        <tr><td>102</td><td>文学部</td><td></td></tr>
                        <tr><td>103</td><td>文学部</td><td>2018-09-10</td></tr>
                        <tr><td>200</td><td>文学部</td><td>2018-09-22</td></tr>
                        <tr><td>201</td><td>工学部</td><td></td></tr>
                        <tr><td>202</td><td>経済学部</td><td>2018-09-25</td></tr>
                    </tbody>
                </table>
            </td>
		</tr>
	</tbody>
</table>

まず、最初にCOUNT関数の挙動について、以下のクエリを用いて簡単に説明する。「COUNT(*)は全行を数える」のに対し、「COUNT(列名)はNULLではない数」を数える。

```sql
SELECT COUNT(*), COUNT(col_1) FROM nulltbl;

-- count(*) count(col1)
-- 3        0
```

上記の挙動を応用し、Studentsテーブルから「全ての学生が提出済みの学部」を抽出する。実際のクエリと取得イメージを以下に示す。

```sql
-- 全ての学生が提出済みの学部を取得するクエリ その1 COUNT関数の利用
SELECT dpt FROM students
GROUP BY dpt
HAVING COUNT(*) = COUNT(sbmt_date);

-- 全ての学生が提出済みの学部を取得するクエリ その2 CASE文の利用
SELECT dpt FROM students
GROUP BY dpt
HAVING COUNT(*) = SUM(CASE WHEN sbmt_date IS NULL THEN 0 ELSE 1 END);
```

<img src="images/NULLを含むデータの取り扱い.png" width=85%>

<div style="page-break-before:always"></div>

##### 特性関数の応用

上述のCASE文を用いたクエリについて、**特性関数**(特定の条件を満たす集合か否かを決める関数)と呼ばれており、以降は以下のテーブルを用いて特性関数の練習を行う。

<table>
    <caption>TestResults</caption>
    <thead>
        <tr>
            <th>student_id</th>
            <th>class</th>
            <th>sex</th>
            <th>score</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>001</td><td>A</td><td>男</td><td>100</td></tr>
        <tr><td>002</td><td>A</td><td>女</td><td>100</td></tr>
        <tr><td>003</td><td>A</td><td>女</td><td>49</td></tr>
        <tr><td>004</td><td>A</td><td>男</td><td>30</td></tr>
        <tr><td>005</td><td>B</td><td>女</td><td>100</td></tr>
        <tr><td>006</td><td>B</td><td>男</td><td>92</td></tr>
        <tr><td>007</td><td>B</td><td>男</td><td>80</td></tr>
        <tr><td>008</td><td>B</td><td>男</td><td>80</td></tr>
        <tr><td>009</td><td>B</td><td>女</td><td>10</td></tr>
        <tr><td>010</td><td>C</td><td>男</td><td>92</td></tr>
        <tr><td>011</td><td>C</td><td>男</td><td>80</td></tr>
        <tr><td>012</td><td>C</td><td>女</td><td>21</td></tr>
        <tr><td>013</td><td>D</td><td>女</td><td>100</td></tr>
        <tr><td>014</td><td>D</td><td>女</td><td>0</td></tr>
        <tr><td>015</td><td>D</td><td>女</td><td>0</td></tr>
    </tbody>
</table>

- 【**練習1**】クラスの75%以上の生徒が80点以上のクラスを選択せよ
- 【**練習2**】50点以上を取った生徒のうち、男子の数が女子の数より多いクラスを選択せよ
- 【**練習3**】女子の平均点が男子の平均点より高いクラスを選択せよ
- 【**練習4**】練習3において男子もしくは女子が一人もいないクラスは取得しない

特に、練習3と4において要件によるがどちらも使いこなせるようにしておきたい。つまり、<font color=red>設計時に要素数が0か、カラムはNULLか、を考慮できる視点を持っておく必要がある</font>。

```sql
-- 【練習1】クラスの75%以上の生徒が80点以上のクラス
SELECT class FROM testresults
GROUP BY class
HAVING COUNT(*) * 0.75 <= SUM(CASE WHEN score >= 80 THEN 1 ELSE 0 END);

-- 【練習2】50点以上を取った生徒のうち、男子の数が女子の数より多いクラス
SELECT class FROM testresults
GROUP BY class
HAVING SUM(CASE WHEN score >= 50 AND sex = '男' 
		THEN 1 
		ELSE 0 END)
	> SUM(CASE WHEN score >= 50 AND sex = '女' 
		THEN 1
		ELSE 0 END);

-- 【練習3】女子の平均点が男子の平均点より高いクラス
SELECT class FROM testresults
GROUP BY class
HAVING AVG(CASE WHEN sex = '女' THEN score ELSE 0 END)
	> AVG(CASE WHEN sex = '男' THEN score ELSE 0 END);

-- 【練習4】練習3において男子もしくは女子が一人もいないクラスは取得しない
SELECT class FROM testresults
GROUP BY class
HAVING AVG(CASE WHEN sex = '女' THEN score ELSE NULL END) -- NULLを返す
	> AVG(CASE WHEN sex = '男' THEN score ELSE NULL END); -- NULLを返す
```

<div style="page-break-before:always"></div>

#### HAVING句で全称量化

<table>
    <caption>Teams</caption>
    <thead>
        <tr>
            <th><u>member</th>
            <th><u>team_id</th>
            <th>status</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>ジョー</td><td>1</td><td>待機</td></tr>
        <tr><td>ケン</td><td>1</td><td>出動中</td></tr>
        <tr><td>ミック</td><td>1</td><td>待機</td></tr>
        <tr><td>カレン</td><td>2</td><td>出動中</td></tr>
        <tr><td>キース</td><td>2</td><td>休暇</td></tr>
        <tr><td>ジャン</td><td>3</td><td>待機</td></tr>
        <tr><td>ハート</td><td>3</td><td>待機</td></tr>
        <tr><td>ディック</td><td>3</td><td>待機</td></tr>
        <tr><td>ベス</td><td>4</td><td>待機</td></tr>
        <tr><td>アレン</td><td>5</td><td>出動中</td></tr>
        <tr><td>ロバート</td><td>5</td><td>休暇</td></tr>
        <tr><td>ケーガン</td><td>5</td><td>待機</td></tr>
    </tbody>
</table>

上記テーブルを用いて「全メンバーが待機中のチーム」を取得するクエリを考える。**①NOT EXISTS**と**②HAVING句**と2つの取得方法があり、<font color=red>パフォーマンスと可読性のトレードオフを持つ</font>。全称量化を存在量化に置き換えた文を以下に示す。

$$
\begin{align*}
全メンバの状態&が「待機」である\\
&\Updownarrow \\
待機中ではないメンバ&がチーム内に1人も存在しない
\end{align*}
$$

具体的なクエリについては下記3つの表現方法がある。

- **その1** NOT EXISTS句を利用
- **その2** HAVING句でSUM関数を利用
- **その3** HAVING句でMINとMAX関数を利用

```sql
-- 全メンバーが待機中のチームのクエリ その1 NOT EXISTS句を利用
SELECT team_id FROM teams T1
WHERE NOT EXISTS(
	SELECT team_id FROM teams T2
	WHERE T1.team_id = T2.team_id AND　T2.status <> '待機'
)
GROUP BY team_id;

-- 全メンバーが待機中のチームのクエリ その2 HAVING句でSUM関数を利用
SELECT team_id FROM teams
GROUP BY team_id
HAVING COUNT(*) = SUM(CASE WHEN status = '待機' THEN 1 ELSE 0 END);

-- 全メンバーが待機中のチームのクエリ その3 HAVING句でMINとMAX関数を利用
SELECT team_id FROM teams
GROUP BY team_id -- HAVING句で集合に1種類しかないことを示す
HAVING MIN(status) = '待機' AND MAX(status) = '待機'; 
```

上記のクエリのうち、その3のMINとMAX関数を用いた方法はインデックスを利用している場合はパフォーマンスにも優れ、また下記クエリのように一覧表示にも活用できる。

```sql
-- 総員スタンバイかどうかをチームごとに一覧表示
SELECT team_id, CASE 
    WHEN MIN(status) = '待機' AND MAX(status) = '待機' THEN '総員スタンバイ'
    ELSE 'メンバーが不足しています'
END AS status
FROM teams
GROUP BY team_id;

-- 出力結果(順番は保証されていない)
-- team_id  status
-- 3        総員スタンバイ
-- 5        メンバーが不足しています
-- 4        総員スタンバイ
-- 2        メンバーが不足しています
-- 1        メンバーが不足しています
```

<div style="page-break-before:always"></div>

#### 一意集合と多重集合

本節では、重複を認めない集合を**一意集合**、重複を認める集合を**多重集合**と呼ぶ。RDBでは多重集合が扱われている。以下のテーブルから「資材がタブっている拠点」を取得する。
　実際のクエリと取得イメージを以下に示す。

<table>
    <caption>Materials</caption>
    <thead>
        <tr>
            <th><u>center</th>
            <th><u>receive_date</th>
            <th>material</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>東京</td><td>2018-04-01</td><td>錫</td></tr>
        <tr><td>東京</td><td>2018-04-12</td><td>亜鉛</td></tr>
        <tr><td>東京</td><td>2018-05-17</td><td>アルミニウム</td></tr>
        <tr><td>東京</td><td>2018-05-20</td><td>亜鉛</td></tr>
        <tr><td>大阪</td><td>2018-04-20</td><td>銅</td></tr>
        <tr><td>大阪</td><td>2018-04-22</td><td>ニッケル</td></tr>
        <tr><td>大阪</td><td>2018-04-29</td><td>鉛</td></tr>
        <tr><td>名古屋</td><td>2018-03-15</td><td>チタン</td></tr>
        <tr><td>名古屋</td><td>2018-04-01</td><td>炭素鋼</td></tr>
        <tr><td>名古屋</td><td>2018-04-24</td><td>炭素鋼</td></tr>
        <tr><td>名古屋</td><td>2018-05-02</td><td>マグネシウム</td></tr>
        <tr><td>名古屋</td><td>2018-05-10</td><td>チタン</td></tr>
        <tr><td>福岡</td><td>2018-05-10</td><td>亜鉛</td></tr>
        <tr><td>福岡</td><td>2018-05-28</td><td>錫</td></tr>
    </tbody>
</table>

<div style="page-break-before:always"></div>

```sql
-- 資材がダブっている拠点を選択する その1 HAVING句の利用
SELECT center FROM materials
GROUP BY center
HAVING COUNT(*) = COUNT(DISTINCT material);

-- 資材がダブっている拠点を選択する その2 EXISTS句の利用
SELECT center FROM materials M1
WHERE EXISTS(
	SELECT center FROM materials M2
	WHERE   M1.center       =  M2.center AND
		M1.material     =  M2.material AND
		M1.receive_date <> M2.receive_date
)
GROUP BY center;
```

<img src="images/拠点ごとの部分集合.png">

また、上記のクエリを応用して、拠点ごとにダブりの有無を出力するクエリも出力可能である。

```sql
-- 資材がダブっているかどうかを拠点ごとに出力する
SELECT center, CASE
	WHEN COUNT(*) = COUNT(DISTINCT material) THEN 'ダブり有り'
	ELSE 'ダブり無し'
END AS status
FROM materials
GROUP BY center;
```

<div style="page-break-before:always"></div>

#### 関係除算でバスケット解析

※【**バスケット分析**】「頻繁に一緒に買われる商品」の法則性を見つける手法。「商品陳列の最適化」や「キャンペーン企画」、「レコメンデーション」など、様々なマーケティング戦略に活用可能。例えば、「おむつとビール」や「スイカと花火」、「お酒とおつまみ」など。

<table>
	<tbody>
		<tr>
			<td>
                <table>
                    <caption>Items</caption>
                    <thead>
                        <tr>
                            <th>item</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr><td>ビール</td></tr>
                        <tr><td>紙オムツ</td></tr>
                        <tr><td>自転車</td></tr>
                    </tbody>
                </table>
            </td>
			<td>
                <table>
                    <caption>ShopItems</caption>
                    <thead>
                        <tr>
                            <th>shop</th>
                            <th>item</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr><td>仙台</td><td>ビール</td></tr>
                        <tr><td>仙台</td><td>紙オムツ</td></tr>
                        <tr><td>仙台</td><td>自転車</td></tr>
                        <tr><td>仙台</td><td>カーテン</td></tr>
                        <tr><td>東京</td><td>ビール</td></tr>
                        <tr><td>東京</td><td>紙オムツ</td></tr>
                        <tr><td>東京</td><td>自転車</td></tr>
                        <tr><td>大阪</td><td>テレビ</td></tr>
                        <tr><td>大阪</td><td>紙オムツ</td></tr>
                        <tr><td>大阪</td><td>自転車</td></tr>
                    </tbody>
                </table>
            </td>
		</tr>
	</tbody>
</table>

上記の「Items」テーブルの全商品を揃えている店舗を選択するクエリを考える。まずは、間違ったSQLを以下に示す。

```sql
-- 【間違ったSQL】ビールと紙オムツと自転車の全てをおいている店舗検索
SELECT DISTINCT shop FROM shopitems
WHERE item IN (SELECT item FROM items); -- OR検索になる

-- 出力結果
-- shop
-- 東京
-- 仙台
-- 大阪 ← 【間違い】ビールは置いていないのに出力される
```

次に、正しいSQLを示す。これは①**剰余を持った除算(INNER JOIN)**と②**厳密な関係除算(LEFT OUTER JOIN)** の二つがある。①は「Itemsテーブルの<u>要素を全て含んでいる</u>」という条件で、②は「Itemsテーブルの<u>要素のみを含んでいる</u>」という条件で取得している。

```sql
-- 【正しいSQL: 剰余を持った除算】ビールと紙オムツと自転車の全てをおいている店舗検索
SELECT shop FROM shopitems SI
INNER JOIN items I ON SI.item = I.item
GROUP BY shop
HAVING COUNT(SI.item) = (SELECT COUNT(*) FROM items); -- 個数が一致している拠点を選ぶ

-- 出力結果
-- shop
-- 東京
-- 仙台

-- 【正しいSQL: 厳密な関係除算】ビールと紙オムツと自転車の全てをおいている店舗検索
SELECT shop FROM shopitems SI
LEFT OUTER JOIN items I ON SI.item = I.item
GROUP BY shop
HAVING COUNT(SI.item) = (SELECT COUNT(*) FROM items) AND 
    COUNT(I.item) = (SELECT COUNT(*) FROM items);

-- 出力結果
-- shop
-- 東京
```

##### 【補足】関係除算について

上記のバスケット解析の演算は一般に「関係除算」と呼ばれ、「$ShopItems ÷ Items$」のように解釈できる。SQLにおいて掛け算に相当するのは「クロス結合」であり、<u>上記2つのSQLとItemsをクロス結合するとShopItemsの部分集合(完全なShopItemsに戻るとは限らない)が得られる</u>。実際に出力結果を比較する。

<img src="images/関係除算.png" width=35%>

```sql
-- 剰余を持った除算
WITH answer1 AS (
	SELECT shop FROM shopitems SI
	INNER JOIN items I ON SI.item = I.item
	GROUP BY shop
	HAVING COUNT(SI.item) = (SELECT COUNT(*) FROM items)
)
SELECT * FROM items CROSS JOIN answer1;

-- 厳密な関係除算
WITH answer2 AS(
	SELECT shop FROM shopitems SI
	LEFT OUTER JOIN items I ON SI.item = I.item
	GROUP BY shop
	HAVING COUNT(SI.item) = (SELECT COUNT(*) FROM items) AND 
	    COUNT(I.item) = (SELECT COUNT(*) FROM items)
)
SELECT * FROM items CROSS JOIN answer2;
```

<table>
    <caption>実際の演算結果</caption>
    <tbody>
        <tr>
            <td>
                <table>
                    <caption>ShopItemsテーブル</caption>
                    <thead>
                        <tr>
                            <th>shop</th>
                            <th>item</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr><td>仙台</td><td>ビール</td></tr>
                        <tr><td>仙台</td><td>紙オムツ</td></tr>
                        <tr><td>仙台</td><td>自転車</td></tr>
                        <tr><td>仙台</td><td>カーテン</td></tr>
                        <tr><td>東京</td><td>ビール</td></tr>
                        <tr><td>東京</td><td>紙オムツ</td></tr>
                        <tr><td>東京</td><td>自転車</td></tr>
                        <tr><td>大阪</td><td>テレビ</td></tr>
                        <tr><td>大阪</td><td>紙オムツ</td></tr>
                        <tr><td>大阪</td><td>自転車</td></tr>
                    </tbody>
                </table>
            </td>
            <td>
                <table>
                    <caption>Items×answer1</caption>
                    <thead>
                        <tr>
                            <th>shop</th>
                            <th>item</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td>東京</td>
                            <td>ビール</td>
                        </tr>
                        <tr>
                            <td>東京</td>
                            <td>紙オムツ</td>
                        </tr>
                        <tr>
                            <td>東京</td>
                            <td>自転車</td>
                        </tr>
                        <tr>
                            <td>仙台</td>
                            <td>ビール</td>
                        </tr>
                        <tr>
                            <td>仙台</td>
                            <td>紙オムツ</td>
                        </tr>
                        <tr>
                            <td>仙台</td>
                            <td>自転車</td>
                        </tr>
                    </tbody>
                </table>
            </td>
            <td>
                <table>
                    <caption>Items×answer2</caption>
                    <thead>
                        <tr>
                            <th>shop</th>
                            <th>item</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td>東京</td>
                            <td>ビール</td>
                        </tr>
                        <tr>
                            <td>東京</td>
                            <td>紙オムツ</td>
                        </tr>
                        <tr>
                            <td>東京</td>
                            <td>自転車</td>
                        </tr>
                    </tbody>
                </table>
            </td>
        </tr>
    </tbody>
</table>

<div style="page-break-before:always"></div>

#### 演習問題

##### 問題6-1 歯抜けを探す〜改良版〜

データの歯抜けを探すクエリについて、歯抜けがある場合は「歯抜けあり」、ない場合は「歯抜けなし」と必ず結果を一行返すように修正せよ。

##### 問題6-1の回答

```sql
-- 【良い例】
SELECT CASE 
    WHEN COUNT(*) <> MAX(seq) THEN '歯抜けあり'
    ELSE '歯抜けなし' 
END AS gap
FROM seqtbl;

-- 【悪い例】パフォーマンスに劣る
SELECT '歯抜けあり' AS gap FROM seqtbl HAVING COUNT(*) <> MAX(seq)
UNION ALL
SELECT '歯抜けなし' AS gap FROM seqtbl HAVING COUNT(*) = MAX(seq);
```

<div style="page-break-before:always"></div>

##### 問題6-2 特性関数の練習

<table>
    <caption>Students</caption>
    <thead>
        <tr>
            <th>学生ID</th>
            <th>学部</th>
            <th>入学日</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>100</td><td>理学部</td><td>2018-10-10</td></tr>
        <tr><td>101</td><td>理学部</td><td>2018-09-22</td></tr>
        <tr><td>102</td><td>文学部</td><td></td></tr>
        <tr><td>103</td><td>文学部</td><td>2018-09-10</td></tr>
        <tr><td>200</td><td>文学部</td><td>2018-09-22</td></tr>
        <tr><td>201</td><td>工学部</td><td></td></tr>
        <tr><td>202</td><td>経済学部</td><td>2018-09-25</td></tr>
    </tbody>
</table>

上記テーブルについて、「全員が9月中に提出済みの学部」を選択するSQLを考えよ。<u>正しいSQLの場合は経済学部のみが取得される。理学部は100番が10月になっているため却下、文学部と工学部は曽本も未提出がいるため却下</u>。

##### 問題6-2の回答

「9月中」という条件はいくつかあるが、以下に2つの回答例を示す。

```sql
-- 全員が9月中に提出済みの学部 その1 BETWEEN句の利用
SELECT dpt FROM students
GROUP BY dpt
HAVING COUNT(*) = SUM(CASE 
	WHEN sbmt_date BETWEEN '2018-09-01' AND '2018-09-30' THEN 1
	ELSE 0
END);

-- 全員が9月中に提出済みの学部 その2 日付データを利用
SELECT dpt FROM students
GROUP BY dpt
HAVING COUNT(*) = SUM(CASE 
	WHEN EXTRACT(YEAR FROM sbmt_date) = 2018
	    AND EXTRACT(MONTH FROM sbmt_date) = 09 THEN 1
	ELSE 0
END);
```

##### 問題6-3 関係除算の改良

<table>
	<tbody>
		<tr>
			<td>
                <table>
                    <caption>Items</caption>
                    <thead>
                        <tr>
                            <th>item</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr><td>ビール</td></tr>
                        <tr><td>紙オムツ</td></tr>
                        <tr><td>自転車</td></tr>
                    </tbody>
                </table>
            </td>
			<td>
                <table>
                    <caption>ShopItems</caption>
                    <thead>
                        <tr>
                            <th>shop</th>
                            <th>item</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr><td>仙台</td><td>ビール</td></tr>
                        <tr><td>仙台</td><td>紙オムツ</td></tr>
                        <tr><td>仙台</td><td>自転車</td></tr>
                        <tr><td>仙台</td><td>カーテン</td></tr>
                        <tr><td>東京</td><td>ビール</td></tr>
                        <tr><td>東京</td><td>紙オムツ</td></tr>
                        <tr><td>東京</td><td>自転車</td></tr>
                        <tr><td>大阪</td><td>テレビ</td></tr>
                        <tr><td>大阪</td><td>紙オムツ</td></tr>
                        <tr><td>大阪</td><td>自転車</td></tr>
                    </tbody>
                </table>
            </td>
		</tr>
	</tbody>
</table>

上記テーブルについて、品物を全て揃えていなかった店舗についても「どれぐらいの品物が不足していたのか」を一覧表示せよ。ここで、店舗の現存在庫数を`my_item_cnt`、店舗の不足商品数を`diff_cnt`とする。

##### 問題6-3の回答

`diff_cnt`はItemsテーブルの行数から各店舗の商品数を引けば良い。ただし、<u>Itemsテーブルに含まれていない商品は除外する必要があるため、**INNER JOINによる内部結合を使う**必要がある</u>。

```sql
SELECT 
	SI.shop,
	COUNT(SI.item) AS my_item_cnt,
	(SELECT COUNT(item) FROM items) - COUNT(SI.item) AS diff_cnt
FROM shopitems SI
INNER JOIN items I ON I.item = SI.item -- 内部結合
GROUP BY SI.shop;
```
