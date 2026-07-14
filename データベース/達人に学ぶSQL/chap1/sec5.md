## EXISTS述語の使い方

<div style="padding: 10px; margin-bottom: 10px; border: 5px double;">
    <h5>学習のポイント</h5>
    <ul>
        <li><font color=red>SQLにおける<b>述語とは真理値を返す関数のこと</b>。</font></li>
        <li><font color=red>全称量化をクエリで表現する場合、EXISTSはパフォーマンス優位、HAVINGは可読性優位になりやすい。</font></li>
        <li>述語論理においてRDBのテーブルは「文(命題)の集合」である</li>
        <li>現在のSQLでは「0階(行)の存在」しか表現出来ない。</li>
        <li>EXISTSだけが他の述語と違って(行の)集合を引数に取り、高階関数(関数を引数に取る関数)の一種とみなせる。</li>
        <li>SQLには全称量子化に相当する演算子がないため、<b>NOT EXISTS</b>で代用する。</li>
        <li><b>列方向の量化</b>はALLやANYを用いる。</li>
    </ul>
</div>


#### はじめに

SQLとRDBを支える基礎理論は「**①集合論(数学の一分野)**」と「**②述語論理(現代論理学)**」の2つから成り立っている。②について、正確には少し範囲を限定した「一階述語論理」が対象となっている。EXISTSは非常に有用な機能であり、具体的には以下の特徴を持つ。

1. 複数行を一単位とみなした高度な条件を記述することができる
2. 相関サブクエリを利用するにも関わらずパフォーマンスが非常に優れる

以下、EXISTSの理論編と実践編を示す。

<div style="page-break-before:always"></div>

#### 理論編

##### 述語とは何か？

<b>述語とは戻り値が真理値(true, false, unknown)になる関数</b>であり、=、<、>、BETWEEN、LIKE、IN、IS NULLなどがある。述語の機能があるからこそテーブルの構造を調べることができる。<font color=red><u>述語論理においてRDBのテーブルは命題の集合(文の集合)と見なすことができる</u></font>。

<img src="images/テーブルは文の集合.png">

##### 存在の階層

EXISTSは=やBETWEENなどの他の述語と違い、「行の集合」を述語の引数に取る。特に、1行を入力とする述語を「一階の述語」、行の集合を入力とする述語を「2階の述語」と呼ぶ。

<img src="images/EXISTSの挙動.png" width=75%>

また、存在の階層で表現する場合、「行」が0階、「テーブル(行の集合)」が1階、「DB(テーブルの集合)」が2階となる。現在のSQLは「データD1を含む**行が存在するか**」という0階の存在しか述語で表現できない。もし将来「データD1を含む**テーブルが存在するか**」という<font color=red>1階の存在(テーブルの存在)を述語で表現できるSQLが作られた場合、データベース分野におけるパラダイムシフトになる可能性がある。</font>

<img src="images/RDBにおける階層の存在.png" width=70%>

##### 全称量化と存在量化

全称量子化と存在量子化のうち、SQLでは存在量子化をサポートしている。

- 【**全称量子化$\forall$**】「全ての$x$が条件$P$を満たす」や「for All $x$, 〜」で表現される。 ← SQLで**サポートされていない。**
- 【**存在量子化$\exists$**】「条件$P$を満たす$x$が少なくとも1つ存在する」や「there Exists $x$ that 〜」で表現される。 ← SQLで**サポートされている。**

ただし、ド・モルガンの法則を用いて全称量子化を表現することができる。

$$
\begin{align*}
\mathbf{\textcolor{red}{\forall}}\hspace{1mm}xPx&= \neg\hspace{1mm}\mathbf{\textcolor{blue}{\exist}}\hspace{1mm}x\hspace{1mm}\neg\hspace{1mm}P x \\
全てのxが条件Pを満たす&=条件Pを満たさないxが存在しない\\ \\
\mathbf{\textcolor{blue}{\exist}}\hspace{1mm}xPx&= \neg\hspace{1mm}\mathbf{\textcolor{red}{\forall}}\hspace{1mm}x\hspace{1mm}\neg\hspace{1mm}P x \\
条件Pを満たすxが存在する&=全てのxが条件Pを満たさないわけではない
\end{align*}
$$

#### 実践編

##### テーブルに存在「しない」データを探す

<table>
    <caption>ミーティング</caption>
    <thead>
    <tr>
        <th>回次</th>
        <th>氏名</th>
    </tr>
    </thead>
    <tbody>
        <tr><td>第1回</td><td>伊藤</td></tr>
        <tr><td>第1回</td><td>水島</td></tr>
        <tr><td>第1回</td><td>坂東</td></tr>
        <tr><td>第2回</td><td>伊藤</td></tr>
        <tr><td>第2回</td><td>宮田</td></tr>
        <tr><td>第3回</td><td>坂東</td></tr>
        <tr><td>第3回</td><td>水島</td></tr>
        <tr><td>第3回</td><td>宮田</td></tr>
    </tbody>
</table>

上記のミーティングの出席者一覧から欠席者一覧を求めることを考える。この場合、「全員出席した場合の集合(DISTINCTを用いたCROSS JOIN)」を使って実際の出席一覧に存在しない(NOT EXISTS)のデータを取得する。

```sql
-- 欠席者だけを求めるクエリ1 存在量化の応用
SELECT DISTINCT M1.meeting, M2.person
FROM ミーティング M1 CROSS JOIN ミーティング M2 -- 全員が出席した場合の仮想の一覧
WHERE NOT EXISTS( -- 実際に出席した人のうち欠席した(存在しない)人を条件として抽出
	SELECT * FROM ミーティング M3
	WHERE M1.meeting = M3.meeting AND M2.person = M3.person
);

-- 欠席者だけを求めるクエリ2 差集合の利用
SELECT DISTINCT M1.meeting, M2.person
FROM ミーティング M1 CROSS JOIN ミーティング M2
EXCEPT SELECT meeting, person FROM ミーティング; -- 差集合演算
```

<div style="page-break-before:always"></div>

##### 全称量化〜肯定↔︎二重否定の変換に慣れよう〜

<table>
    <caption>テスト結果</caption>
    <thead>
        <tr>
            <th>生徒ID</th>
            <th>科目</th>
            <th>得点</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>100</td><td>算数</td><td>100</td></tr>
        <tr><td>100</td><td>国語</td><td>80</td></tr>
        <tr><td>100</td><td>理科</td><td>80</td></tr>
        <tr><td>200</td><td>算数</td><td>80</td></tr>
        <tr><td>200</td><td>国語</td><td>95</td></tr>
        <tr><td>300</td><td>算数</td><td>40</td></tr>
        <tr><td>300</td><td>国語</td><td>90</td></tr>
        <tr><td>300</td><td>社会</td><td>55</td></tr>
        <tr><td>400</td><td>算数</td><td>80</td></tr>
    </tbody>
</table>

上記のテスト結果から「全ての教科が50点以上の生徒」を取得する。SQLでは全称量子化は使えないため、以下のように変換する。

$$
\begin{align*}
全教科が&50点以上の生徒\Leftrightarrow 50点未満である教科が一つもない生徒
\end{align*}
$$

```sql
-- 全教科が50点以上の生徒　⇄　50点未満である教科が一つもない生徒
SELECT DISTINCT T1.生徒ID FROM テスト結果 T1
WHERE NOT EXISTS(
    SELECT T2.生徒ID FROM テスト結果 T2
    WHERE T1.生徒ID = T2.生徒ID AND 
        T2.得点 < 50
);
```

<div style="page-break-before:always"></div>

さらに「算数の点数が80点以上」かつ「国語の点数が50点以上」を満たす生徒の取得を考える。基礎のSQL①と応用のSQL②を以下に示す。②は①の強化版である。

- **①のクエリの特徴**
  - DISTINCTが必要
  - 算数と国語の点数がない生徒も抽出される
- **②のクエリの特徴**
  - DISTINCTが不要(学生IDでGROUP BYしているため)
  - 算数と国語の点数がない生徒は抽出されない(INで絞り込み、HAVING句で個数を調べているため)

```sql
-- ①【基礎】算数の点数が80点以上かつ国語の点数が50点以上の生徒
-- ※算数と国語のテスト結果を持っていない学生も「抽出される」
SELECT DISTINCT T1.学生ID FROM テスト結果 T1
WHERE T1.科目 IN ('算数', '国語') AND -- 算数と国語に絞り込み
	NOT EXISTS(
	    SELECT T2.学生ID FROM テスト結果 T2
	    WHERE T1.学生ID = T2.学生ID AND 
	        1 = CASE -- 特製関数(ある行が条件を満たすかどうかを判別する関数)
	            WHEN 科目 = '算数' AND 得点 < 80 THEN 1
	            WHEN 科目 = '国語' AND 得点 < 50 THEN 1
	            ELSE 0 
	        END
	);

-- ②【応用】算数の点数が80点以上かつ国語の点数が50点以上の生徒
-- ※算数と国語のテスト結果を持っていない学生は「抽出されない」
SELECT T1.学生ID FROM テスト結果 T1
WHERE T1.科目 IN ('算数', '国語') AND -- 算数と国語に絞り込み
	NOT EXISTS(
	    SELECT T2.学生 FROM テスト結果 T2
	    WHERE T1.学生ID = T2.学生ID AND 
	        1 = CASE -- 特製関数(ある行が条件を満たすかどうかを判別する関数)
	            WHEN 科目 = '算数' AND 得点 < 80 THEN 1
	            WHEN 科目 = '国語' AND 得点 < 50 THEN 1
	            ELSE 0 
	        END
	)
GROUP BY 学生ID
HAVING COUNT(*) = 2; -- 算数と国語のテスト結果を持っていない学生は抽出しない
```

<div style="page-break-before:always"></div>

##### 全称量化〜集合 vs. 述語すごいのはどっち？〜

<table>
    <caption>プロジェクト一覧</caption>
    <thead>
        <tr>
            <th>プロジェクトID</th>
            <th>工程番号</th>
            <th>ステータス</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>AA100</td><td>0</td><td>完了</td></tr>
        <tr><td>AA100</td><td>1</td><td>待機</td></tr>
        <tr><td>AA100</td><td>2</td><td>待機</td></tr>
        <tr><td>B200</td><td>0</td><td>待機</td></tr>
        <tr><td>B200</td><td>1</td><td>待機</td></tr>
        <tr><td>CS300</td><td>0</td><td>完了</td></tr>
        <tr><td>CS300</td><td>1</td><td>完了</td></tr>
        <tr><td>CS300</td><td>2</td><td>待機</td></tr>
        <tr><td>CS300</td><td>3</td><td>待機</td></tr>
        <tr><td>DY400</td><td>0</td><td>完了</td></tr>
        <tr><td>DY400</td><td>1</td><td>完了</td></tr>
        <tr><td>DY400</td><td>2</td><td>完了</td></tr>
    </tbody>
</table>

上記のテーブルから工程番号1まで完了しているプロジェクトを選択するクエリを考える。クエリは2種類あり、「HAVING句を用いたクエリ」と「NOT EXISTSを用いたクエリ」が考えられる。NOT EXISTSのクエリについては以下の全称命題をクエリで表現している。

$$
\begin{align*}
\mathbf{\textcolor{red}{\forall}}\hspace{1mm}xPx&= \neg\hspace{1mm}\mathbf{\textcolor{blue}{\exist}}\hspace{1mm}x\hspace{1mm}\neg\hspace{1mm}P x
\end{align*}
$$
$$
\begin{align*}
&任意のプロジェクトにおいて、全ての工程が\\
&工程番号が1以下ならば完了であり、1より大きければ待機である\\
&を満たす。\\
&\Leftrightarrow \\
&任意のプロジェクトにおいて、\\
&工程番号が1以下ならば完了であり、1より大きければ待機である\\
&を満たさないプロジェクト内の工程は存在しない。
\end{align*}
$$

- 【**HAVING句のクエリ**】各プロジェクトIDごとに工程番号が1以下で「完了」の行数と1より大きくて「待機」の行数の総和がグループ全体の行数と一致するプロジェクトIDを選択する。
  - **メリット** 可読性に優れる。
  - **デメリット** NOT EXISTSと比較してパフォーマンスに劣る。
- 【**NOT EXISTSのクエリ**】任意のプロジェクトにおいて、「工程番号が1以下ならば完了であり、1より大きければ待機である」を満たさない工程が1つも存在しないプロジェクトIDを選択する。
  - **メリット** パフォーマンスに優れる。取得(SELECT)できる情報が多い(GROUP BYがないため)。
  - **デメリット** HAVING句と比較して可読性が大きく下がる。

```sql
-- 【HAVINGバージョン】工程番号1まで完了しているプロジェクトを選択するクエリ
SELECT プロジェクトID FROM プロジェクト一覧
GROUP BY プロジェクトID
HAVING COUNT(*) = SUM(CASE -- 「個数」と「状態の総和」が一致するかどうか
    WHEN プロジェクトID <= 1 AND ステータス = '完了' THEN 1
    WHEN プロジェクトID  > 1 AND ステータス = '待機' THEN 1
    ELSE 0
END);

-- 【GROUP BYバージョン】工程番号1まで完了しているプロジェクトを選択するクエリ
SELECT * FROM プロジェクト一覧 P1
WHERE NOT EXISTS(
    SELECT ステータス FROM プロジェクト一覧 P2
    WHERE P1.プロジェクトID = P2.プロジェクトID AND
        P2.ステータス <> CASE 
            WHEN P2.工程番号 <= 1 THEN '完了'
            ELSE '待機'
        END
);
```

<div style="page-break-before:always"></div>

##### 列に対する量化〜オール1の行を探せ〜

<table>
    <caption>列持ちテーブル</caption>
    <thead>
        <tr>
            <th>項目</th>
            <th>列1</th>
            <th>列2</th>
            <th>列3</th>
            <th>列4</th>
            <th>列5</th>
            <th>列6</th>
            <th>列7</th>
            <th>列8</th>
            <th>列9</th>
            <th>列10</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>A</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
        <tr><td>B</td><td>3</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
        <tr><td>C</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td><td>1</td></tr>
        <tr><td>D</td><td></td><td></td><td>9</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr>
        <tr><td>E</td><td></td><td>3</td><td></td><td>1</td><td>9</td><td></td><td></td><td>9</td><td></td><td></td></tr>
    </tbody>
</table>

上記の列持ちテーブルを用いて①オール1の行、②少なくとも1つは9の行、③全てがNULLの行を満たす行をそれぞれ取得する。①は<font color=red>ALL述語</font>、②はANY述語とIN、<font color=red>③はCOALESCE関数</font>を用いる。<u>ALLやANYは量化の機能</u>である。

```sql
-- オール1の行(=項目C)
SELECT * FROM 列持ちテーブル
WHERE 1 = ALL(array[col1, col2, col3, col4, col5, col6, col7, col8, col9, col10]);

-- 【解法1】少なくとも1つは9の行(=項目DとE)
SELECT * FROM 列持ちテーブル
WHERE 9 = ANY(array[col1, col2, col3, col4, col5, col6, col7, col8, col9, col10]);

-- 【解法2】少なくとも1つは9の行(=項目DとE)
SELECT * FROM 列持ちテーブル
WHERE 9 IN(col1, col2, col3, col4, col5, col6, col7, col8, col9, col10);

-- 全てがNULLの行(=項目A)
SELECT * FROM 列持ちテーブル
WHERE COALESCE(col1, col2, col3, col4, col5, col6, col7, col8, col9, col10) IS NULL;
```

<div style="page-break-before:always"></div>

#### 演習問題

##### 問題5-1

<table>
  <thead>
    <tr>
      <th><u>key</th>
      <th><u>i</th>
      <th>val</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>A</td><td>1</td><td></td></tr>
    <tr><td>A</td><td>2</td><td></td></tr>
    <tr><td>A</td><td>3</td><td></td></tr>
    <tr><td></td><td>...</td><td></td></tr>
    <tr><td>B</td><td>1</td><td>3</td></tr>
    <tr><td>B</td><td>2</td><td></td></tr>
    <tr><td>B</td><td>3</td><td></td></tr>
    <tr><td></td><td>...</td><td></td></tr>
    <tr><td>C</td><td>1</td><td>1</td></tr>
    <tr><td>C</td><td>2</td><td>1</td></tr>
    <tr><td>C</td><td>3</td><td>1</td></tr>
    <tr><td></td><td>...</td><td></td></tr>
  </tbody>
</table>

上記の行持ちテーブルからオール1のkeyを取得せよ。

##### 問題5-1の回答

$$
\begin{align*}
全ての行につ&いてval=1である。\\
&\Leftrightarrow \\
val=1ではない&行が1つも存在しない。
\end{align*}
$$

```sql
-- 不正解 → AとCが抽出される。
SELECT DISTINCT key FROM arraytbl2 AT1
WHERE NOT EXISTS(
	SELECT * FROM arraytbl2 AT2
	WHERE AT1.key = AT2.key AND
		AT2.val <> 1
);
```
```sql
-- 回答例1(EXISTS句) → Cが抽出される。
SELECT DISTINCT key FROM arraytbl2 AT1
WHERE NOT EXISTS(
	SELECT * FROM arraytbl2 AT2
	WHERE AT1.key = AT2.key AND
		(AT2.val <> 1 OR AT2.val IS NULL)
);

-- 回答例2(ALL述語) → Cが抽出される。
SELECT DISTINCT key FROM arraytbl2 AT1 -- DISTINCTがないと10個出力される。
WHERE 1 = ALL(
	SELECT AT2.val FROM arraytbl2 AT2
	WHERE AT1.key = AT2.key
);

-- 回答例3(HAVING句) → Cが抽出される。
SELECT key FROM arraytbl2
GROUP BY key
HAVING SUM(CASE WHEN val = 1 THEN 1 ELSE 0 END) = 10;
```

<div style="page-break-before:always"></div>

##### 問題5-2

以下のクエリをALL述語で書き換えよ。

```sql
SELECT * FROM プロジェクト一覧 P1
WHERE NOT EXISTS(
    SELECT ステータス FROM プロジェクト一覧 P2
    WHERE P1.プロジェクトID = P2.プロジェクトID AND
        P2.ステータス <> CASE 
            WHEN P2.工程番号 <= 1 THEN '完了'
            ELSE '待機'
        END
);
```

##### 問題5-2の回答

```sql
SELECT * FROM プロジェクト一覧 P1
WHERE 1 = ALL(
    SELECT CASE 
        WHEN 工程番号 <= 1 AND ステータス = '完了' THEN 1
        WHEN 工程番号  > 1 AND ステータス = '待機' THEN 1
        ELSE 0 
    END
    FROM プロジェクト一覧 P2
    WHERE P1.プロジェクトID = P2.プロジェクトID
);

-- 出力結果
project_id  step_nbr    status
CS300       0           完了
CS300       1           完了
CS300       2           待機
CS300       3           待機
```

<div style="page-break-before:always"></div>

##### 問題5-3

1〜100の数値をもつテーブルから素数を求めるクエリを考えよ。

##### 問題5-3の回答

考え方としては「1とその数以外に割り切れる自然数が存在しない」⇄「1とその数以外に割り切れる自然が一つ以上存在する」という量化を考える。具体的な条件としては以下の通りである。

```sql
SELECT num FROM numbers N1
WHERE N1.num > 1 AND
	NOT EXISTS(
		SELECT num FROM numbers N2
		WHERE N2.num <> 1 AND
			N1.num <= N2.num / 2 AND -- 約数は自分の半分以下にしか存在しない
			mod(N1.num, N2.num) = 0 -- 剰余を求める
	);
```

