## CASE式のすすめ

<div style="padding: 10px; margin-bottom: 10px; border: 5px double;">
    <h5>学習のポイント</h5>
    <ul>
        <li><b>CASE式は複数のSQLを1つにまとめ、可読性とパフォーマンスを向上させる。</b></li>
        <li>CASE式は単純CASE式と検索CASE式の2つがあり、基本的には検索CASE式を使う。</li>
        <li>CASE式は列名や定数をかける場所には常に書くことができ、具体的にはSELECT句、WHERE句、GROUP BY句、HAVING句、ORDER BY句、CHECK制約の中、関数の引数、述語の引数、CASE式の中などがある。</li>
        <li><font color=red>HANVING句で条件分岐させるのは素人のやること。プロはSELECT句で分岐させる。</font></li>
        <li>GROUP BY句でCASE式を使うことで、集約単位となるコードや階級を柔軟に設定できる。</li>
        <li><font color=red>集約関数の中に使うことで行持ちから列持ちへの水平展開も自由自在</font></li>
    </ul>
</div>

<div style="page-break-before:always"></div>

#### CASE式の構文

- CASE式は「文」ではなく「式」であり、単純CASE式と検索CASE式の2つに分けられる。
- CASE式は短絡評価であり、真(TRUE)になるWHENが見つかった時点で打ち切られて残りのWHEN句は無視される(評価されない)

```sql
-- 単純CASE文
CASE sex
    WHEN '1' THEN '男'
    WHEN '2' THEN '女'
    ELSE          'その他'
END

-- 検索CASE文
CASE
    WHEN sex = '1' THEN '男'
    WHEN sex = '2' THEN '女'
    ELSE                'その他'
END
```

- **【注意点1】各分岐が買えるデータ型を統一する**
全ての分岐においてCASE式の戻り値のデータ型は統一していなければならない。
- **【注意点2】ENDの書き忘れに注意**
ENDは必須なので書き忘れがないように注意する。
- **【注意点3】ELSE句は必ず書こう**
<font color=red>ELSEがないと暗黙的に「NULL」を返すため、<b>明示的にELSE句を書く癖</b>をつけておく。</font>

<div style="page-break-before:always"></div>

#### 「既存コード体系」を「新しい体系」に変換して集計する

非定型的な集計を行う業務では「既存のコード体系」を「分析用のコード体系」に変換して集計したいという要件がしばしば発生する。以下に都道府県の人口を地方単位で集計する例を示す。解決策としては**ビューの定義による集計**と**CASE式での集計**の2つがある。
　<font color=red>ビューの定義の場合、ユーザから基底テーブルを隠蔽できるが、「地方コード」などの集計に必要な列を追加する必要があり、アドホックな変更が困難という特徴がある</font>。一方、<font color=red>CASE式での集計の場合、可読性に優れるが、OracleやSQLServerなどの一部のDBMSでは構文エラーになる</font>。

<table>
    <tr>
        <td>
            <table>
                <caption>都道府県</caption>
                <thead>
                    <tr>
                        <th><u>都道府県名</th>
                        <th>人口</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>徳島</td><td>100</td></tr>
                    <tr><td>香川</td><td>200</td></tr>
                    <tr><td>愛媛</td><td>150</td></tr>
                    <tr><td>高知</td><td>200</td></tr>
                    <tr><td>福岡</td><td>300</td></tr>
                    <tr><td>佐賀</td><td>100</td></tr>
                    <tr><td>長崎</td><td>200</td></tr>
                    <tr><td>東京</td><td>400</td></tr>
                    <tr><td>群馬</td><td>50</td></tr>
                </tbody>
            </table>
        </td>
        <td>
            <table>
                <caption>集計結果</caption>
                <thead>
                    <tr>
                        <th><u>地方名</th>
                        <th>人口</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>四国</td><td>650</td></tr>
                    <tr><td>九州</td><td>600</td></tr>
                    <tr><td>その他</td><td>450</td></tr>
                </tbody>
            </table>
        </td>
    </tr>
</table>

```sql
-- ##### 集計結果 #####
SELECT CASE WHEN 都道府県名 = '徳島' THEN '四国'
        WHEN 都道府県名 = '香川' THEN '四国'
        WHEN 都道府県名 = '愛媛' THEN '四国'
        WHEN 都道府県名 = '高知' THEN '四国'
        WHEN 都道府県名 = '福岡' THEN '九州'
        WHEN 都道府県名 = '佐賀' THEN '九州'
        WHEN 都道府県名 = '長崎' THEN '九州'
        ELSE 'その他' END AS 地方名,
    SUM(population) as 人口
FROM 都道府県 GROUP BY 地方名;
```

<div style="page-break-before:always"></div>

#### 異なる条件の集計を1つのSQLで行う

<table>
    <tr>
        <td>
            <table>
                <caption>都道府県</caption>
                <thead>
                    <tr>
                    <th><u>都道府県名</th>
                    <th><u>性別(1: 男、2: 女)</th>
                    <th>人口</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>徳島</td><td>1</td><td>60</td></tr>
                    <tr><td>徳島</td><td>2</td><td>40</td></tr>
                    <tr><td>香川</td><td>1</td><td>100</td></tr>
                    <tr><td>香川</td><td>2</td><td>100</td></tr>
                    <tr><td>愛媛</td><td>1</td><td>100</td></tr>
                    <tr><td>愛媛</td><td>2</td><td>50</td></tr>
                    <tr><td>高知</td><td>1</td><td>100</td></tr>
                    <tr><td>高知</td><td>2</td><td>100</td></tr>
                    <tr><td>福岡</td><td>1</td><td>100</td></tr>
                    <tr><td>福岡</td><td>2</td><td>200</td></tr>
                    <tr><td>佐賀</td><td>1</td><td>20</td></tr>
                    <tr><td>佐賀</td><td>2</td><td>80</td></tr>
                    <tr><td>長崎</td><td>1</td><td>125</td></tr>
                    <tr><td>長崎</td><td>2</td><td>125</td></tr>
                    <tr><td>東京</td><td>1</td><td>250</td></tr>
                    <tr><td>東京</td><td>2</td><td>150</td></tr>
                </tbody>
            </table>
        </td>
        <td>
            <table>
                <caption>集計結果</caption>
                <thead>
                    <tr>
                        <th><u>地方名</th>
                        <th>人口</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>四国</td><td>650</td></tr>
                    <tr><td>九州</td><td>600</td></tr>
                    <tr><td>その他</td><td>450</td></tr>
                </tbody>
            </table>
        </td>
    </tr>
</table>

```sql
-- 都道府県の男女別の人口
SELECT pref_name as 都道府県名,
    SUM( CASE WHEN sex = '1' THEN population ELSE 0 END) AS 男, --男性の人口
    SUM( CASE WHEN sex = '2' THEN population ELSE 0 END) AS 女  --女性の人口
FROM 都道府県
GROUP BY 都道府県名;
```

#### 条件を分岐させたUPDATE

以下の給与改定前のテーブルについて条件1と2を適用する。
- 【**条件1**】現在の給料が30万円以上の社員は、10%の減給とする。
- 【**条件2**】現在の給料が25万円以上28万円未満の社員は、20%の昇給とする。

<font color=red>ここで注意が必要なのは<b>条件1と2を同時に実行すれば問題ないが、それぞれ独立して順番に実行すると異なる結果が得られる</b>ということである。</font>クエリで表現すると以下の通りである。

<table>
    <tr>
        <td>
            <table>
                <caption>給与改定前</caption>
                <thead>
                    <tr>
                        <th><u>氏名</th>
                        <th>給与</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>相田</td><td>300,000</td></tr>
                    <tr><td>神崎</td><td>270,000</td></tr>
                    <tr><td>木村</td><td>220,000</td></tr>
                    <tr><td>斉藤</td><td>290,000</td></tr>
                </tbody>
            </table>
        </td>
        <td>
            <table>
                <caption>給与改定後</caption>
                <thead>
                    <tr>
                        <th><u>氏名</th>
                        <th>給与</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>相田</td><td>270,000(減給)</td></tr>
                    <tr><td>神崎</td><td>324,000(昇給)</td></tr>
                    <tr><td>木村</td><td>220,000</td></tr>
                    <tr><td>斉藤</td><td>290,000</td></tr>
                </tbody>
            </table>
        </td>
    </tr>
</table>


```sql
-- 条件1と条件2の合わせ技
UPDATE Personnel
    SET salary = CASE 
        WHEN salary >= 300000 THEN salary * 0.9 -- 条件1
        WHEN salary >= 250000 AND salary < 280000 THEN salary * 1.2 -- 条件2
        ELSE salary
    END;
```

上記SQLについて、OracleやSQL Serverは実行可能であるが、**PostgreSQLやMySQLでは主キー重複によるエラーが発生するため、遅延制約(DEFERABLE)オプションをつける必要がある**。

```sql
-- 例】遅延制約の設定
CREATE TABLE SomeTable
(
  p_key CHAR(1) PRIMARY KEY DEFERRABLE, -- 遅延制約(DEFERABLE)
  col_1 INTEGER NOT NULL, 
  col_2 CHAR(2) NOT NULL
);
```

<div style="page-break-before:always"></div>

#### テーブル同士のマッチング(行持ち→列持ちへの変換)

講座一覧と月々の開講講座の情報をもとに、CASE式を用いてテーブルのクロス表を作成する。クロス表の作成にはINとEXISTS、外部結合の3通りの方法があるが、今回はINとEXISTSの2つの方法で出力する方法を考える。
　<font color=red>INよりEXISTSの方がパフォーマンスに優れるため、<b>レコード数が多い場合はEXISTSの方が優位性がある</b>。</font>

<table>
    <tr>
        <td>
            <table>
                <caption>講座一覧</caption>
                <thead>
                    <tr>
                        <th><u>講座ID</th>
                        <th>講座名</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>1</td><td>経理入門</td></tr>
                    <tr><td>2</td><td>財務知識</td></tr>
                    <tr><td>3</td><td>簿記検定開講講座</td></tr>
                    <tr><td>4</td><td>税理士</td></tr>
                </tbody>
            </table>
        </td>
        <td>
            <table>
                <caption>月々の開講講座</caption>
                <thead>
                    <tr>
                        <th><u>年月</th>
                        <th><u>講座ID</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>201806</td><td>1</td></tr>
                    <tr><td>201806</td><td>3</td></tr>
                    <tr><td>201806</td><td>4</td></tr>
                    <tr><td>201807</td><td>4</td></tr>
                    <tr><td>201807</td><td>2</td></tr>
                    <tr><td>201807</td><td>4</td></tr>
                </tbody>
            </table>
        </td>
        <td>
            <table>
                <caption>期待する出力結果(クロス表)</caption>
                <thead>
                    <tr>
                        <th><u>口座名</th>
                        <th>6月</th>
                        <th>7月</th>
                        <th>8月</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>経理入門</td><td>○</td><td>×</td><td>×</td></tr>
                    <tr><td>財務知識</td><td>×</td><td>×</td><td>○</td></tr>
                    <tr><td>簿記検定</td><td>○</td><td>×</td><td>×</td></tr>
                    <tr><td>税理士</td><td>○</td><td>○</td><td>○</td></tr>
                </tbody>
            </table>
        </td>
    </tr>
</table>

<div style="page-break-before:always"></div>

```sql
-- IN述語の利用
SELECT
    講座名,
    CASE
        WHEN 講座ID = IN (SELECT 講座ID FROM 月々の開講講座 WHERE 年月 = '201806') THEN ○
        ELSE ×
    END AS '6月',
    CASE
        WHEN 講座ID = IN (SELECT 講座ID FROM 月々の開講講座 WHERE 年月 = '201807') THEN ○
        ELSE ×
    END AS '7月',
    CASE
        WHEN 講座ID = IN (SELECT 講座ID FROM 月々の開講講座 WHERE 年月 = '201808') THEN ○
        ELSE ×
    END AS '8月',
FROM 講座一覧;

-- EXISTS述語の利用
SELECT
    講座名,
    CASE
        WHEN CM.講座ID = EXISTS (
            SELECT 講座ID FROM 月々の開講講座 OC
            WHERE OC.年月 = '201806' AND OC.講座ID = CM.講座ID
        ) THEN ○
        ELSE ×
    END AS '6月',
    CASE
        WHEN CM.講座ID = EXISTS (
            SELECT 講座ID FROM 月々の開講講座 OC
            WHERE OC.年月 = '201807' AND OC.講座ID = CM.講座ID
        ) THEN ○
        ELSE ×
    END AS '7月',
    CASE
        WHEN CM.講座ID = EXISTS (
            SELECT 講座ID FROM 月々の開講講座 OC
            WHERE OC.年月 = '201808' AND OC.講座ID = CM.講座ID
        ) THEN ○
        ELSE ×
    END AS '8月',
FROM 講座一覧 CM;
```

<div style="page-break-before:always"></div>

#### CASE式の中で集約関数を使う

以下の部活動の表を用いて、次の条件を満たすクエリを発行する。
- 【**条件1**】1つだけのクラブに所属している学生については、そのクラブIDを取得する。
- 【**条件2**】複数のクラブを掛け持ちしている学生については、主なクラブIDを取得する。

<table>
    <caption>学生所属クラブ一覧</caption>
    <tbody>
        <tr>
            <th><u>学生番号</th>
            <th><u>クラブID</th>
            <th>クラブ名</th>
            <th>メインクラブフラグ</th>
        </tr>
        <tr>
            <td>100</td>
            <td>1</td>
            <td>野球</td>
            <td>Y</td>
        </tr>
        <tr>
            <td>100</td>
            <td>2</td>
            <td>吹奏楽</td>
            <td>N</td>
        </tr>
        <tr>
            <td>200</td>
            <td>2</td>
            <td>吹奏楽</td>
            <td>N</td>
        </tr>
        <tr>
            <td>200</td>
            <td>3</td>
            <td>バドミントン</td>
            <td>Y</td>
        </tr>
        <tr>
            <td>200</td>
            <td>4</td>
            <td>サッカー</td>
            <td>N</td>
        </tr>
        <tr>
            <td>300</td>
            <td>4</td>
            <td>サッカー</td>
            <td>N</td>
        </tr>
        <tr>
            <td>400</td>
            <td>5</td>
            <td>水泳</td>
            <td>N</td>
        </tr>
        <tr>
            <td>500</td>
            <td>6</td>
            <td>囲碁</td>
            <td>N</td>
        </tr>
    </tbody>
</table>

```sql
-- 【条件1】1つだけのクラブに所属している学生については、そのクラブIDを取得する。
SELECT 学生番号, MAX(クラブID) AS メインクラブ
FROM 学生所属クラブ一覧
GROUP BY 学生番号 HAVING COUNT(*) = 1;

-- 【条件2】複数のクラブを掛け持ちしている学生については、主なクラブIDを取得する。
SELECT 学生番号, クラブID AS メインクラブ
FROM 学生所属クラブ一覧 WHERE メインクラブフラグ = 'Y';

-- 【条件1と2の合わせ技】
SELECT 学生番号,
    CASE 
        WHEN COUNT(*) = 1 THEN クラブID -- 【条件1】
        ELSE MAX(CASE WHEN メインクラブフラグ = 'Y' THEN クラブID ELSE NULL END) -- 【条件2】
    END AS メインクラブ
FROM 学生所属クラブ一覧
GROUP BY 学生番号
```

<div style="page-break-before:always"></div>

#### 演習問題

##### 問題1-1

以下のサンプルデータからxとyの最大値を取得するSQLを作成せよ。また、3列以上にも拡張できるように、xとyとzの最大値を求めるSQLを作成せよ。

<table>
    <caption>サンプルデータ</caption>
	<tbody>
		<tr>
			<th><u>key</th>
			<th>x</th>
			<th>y</th>
			<th>z</th>
		</tr>
		<tr>
			<td>A</td>
			<td>1</td>
			<td>2</td>
			<td>3</td>
		</tr>
		<tr>
			<td>B</td>
			<td>5</td>
			<td>5</td>
			<td>2</td>
		</tr>
		<tr>
			<td>C</td>
			<td>4</td>
			<td>7</td>
			<td>1</td>
		</tr>
		<tr>
			<td>D</td>
			<td>3</td>
			<td>3</td>
			<td>8</td>
		</tr>
	</tbody>
</table>

##### 問題1-1の回答

```sql
-- xとyの最大値取得
SELECT key,
	CASE WHEN x <= y THEN y ELSE x END AS GREATEST
FROM greatests;

-- xとyとzの最大値取得
SELECT key,
	CASE WHEN x <= y THEN CASE
		WHEN y <= z THEN z
		ELSE y END
	ELSE x END AS GREATEST
FROM greatests;

-- 【別解】列持ち→行持ちに変換して最大値取得
SELECT key, MAX(val) AS GREATEST
FROM (
	SELECT key, x AS val FROM greatests UNION ALL
	SELECT key, y AS val FROM greatests UNION ALL
	SELECT key, z AS val FROM greatests
)
GROUP BY key;
```

<div style="page-break-before:always"></div>

##### 問題1-2

行持ちから列持ちに変換する。男女別で全国の人口を出力し、また徳島、香川、愛媛、高知それぞれの人口を出力した後、四国として合計した人口も出力せよ。

<table>
    <caption>都道府県</caption>
    <thead>
        <tr>
        <th><u>pref_name</th>
        <th><u>sex</th>
        <th>population</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>徳島</td><td>1</td><td>60</td></tr>
        <tr><td>徳島</td><td>2</td><td>40</td></tr>
        <tr><td>香川</td><td>1</td><td>100</td></tr>
        <tr><td>香川</td><td>2</td><td>100</td></tr>
        <tr><td>愛媛</td><td>1</td><td>100</td></tr>
        <tr><td>愛媛</td><td>2</td><td>50</td></tr>
        <tr><td>高知</td><td>1</td><td>100</td></tr>
        <tr><td>高知</td><td>2</td><td>100</td></tr>
        <tr><td>福岡</td><td>1</td><td>100</td></tr>
        <tr><td>福岡</td><td>2</td><td>200</td></tr>
        <tr><td>佐賀</td><td>1</td><td>20</td></tr>
        <tr><td>佐賀</td><td>2</td><td>80</td></tr>
        <tr><td>長崎</td><td>1</td><td>125</td></tr>
        <tr><td>長崎</td><td>2</td><td>125</td></tr>
        <tr><td>東京</td><td>1</td><td>250</td></tr>
        <tr><td>東京</td><td>2</td><td>150</td></tr>
    </tbody>
</table>

##### 問題1-2の回答

```sql
SELECT 
	MAX(CASE WHEN sex = '1' THEN '男性' ELSE '女性' END),
	SUM(population),
	SUM(CASE WHEN pref_name = '徳島' THEN population ELSE 0 END) AS 徳島,
	SUM(CASE WHEN pref_name = '香川' THEN population ELSE 0 END) AS 香川,
	SUM(CASE WHEN pref_name = '愛媛' THEN population ELSE 0 END) AS 愛媛,
	SUM(CASE WHEN pref_name = '高知' THEN population ELSE 0 END) AS 高知,
	SUM(CASE WHEN pref_name IN ('徳島', '香川', '愛媛', '高知') THEN population ELSE 0 END) AS 四国
FROM 都道府県
GROUP BY sex;
```

##### 問題1-3

問題1-1において、keyの順番を「B-A-D-C」の順番に並び替えるクエリを考えてください。

##### 問題1-3の回答

```sql
-- B-A-D-Cの順番に並び替えるクエリ
SELECT key,
	CASE 
	    WHEN key = 'B' THEN 1
	    WHEN key = 'A' THEN 2
	    WHEN key = 'D' THEN 3
	    WHEN key = 'C' THEN 4
	    ELSE NULL
	END AS sort_key, -- keyの順番を変えるための工夫
	CASE WHEN x <= y THEN CASE
		WHEN y <= z THEN z
		ELSE y END
	ELSE x END AS GREATEST
FROM greatests
ORDER BY sort_key;
```