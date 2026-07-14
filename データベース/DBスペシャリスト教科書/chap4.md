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
        content: counter(chapter) "章 ";
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

# SQL

## SQL

### SQLとは

SQL(Structured Query Language)の起源は1970年代にIBMが開発したSEQUEL(Structured English Query Language)と言われており、SQLはさまざまなベンダが協力して発展させていった言語である。Oracleなどの一部ベンダで独自機能があったこともあり、仕様の混乱が起きないように標準規格として**標準SQL**が定められた。SQLには大きく以下の3種類ある。

- **SQL Data Definition Language(DDL)** DBを定義するための言語。テーブルやビュー、インデックスなどを「定義」する。
- **SQL Data Control Language(DCL)** DBを制御するための言語。トランザクションの開始や終了、アクセス権限の制御などがある。
- **SQL Data Manipulation Language(DML)** DBのデータを操作するための言語。データの追加(Create)/参照(Read)/更新(Update)/削除(Delete)を行う。

### SQL-DDL(Data Definition Language)

データベースのデータの定義を扱う。作成は`CREATE`、削除は`DROP`、変更は`ALTER`の3つの操作があり、操作対象としてはデータベース、テーブル、ビュー、インデックス、ストアドプロシージャなどが挙げられる。

#### 【操作例】

```sql
-- データベース定義
CREATE DATABASE SomeDataBase;

-- ビューの定義
CREATE VIEW 営業部社員 (社員番号, 社員名) AS
    SELECT 社員番号, 社員名 FROM 社員 WHERE 所属 = '営業部';

-- インデックス(索引)
CREATE INDEX family_name ON Employee(last_name)

```

```sql
-- テーブル作成
CREATE TABLE SomeTable(
    -- 列名 データ型 [列の制約]
    id INTEGER PRIMARY KEY      -- 主キー制約,
    name VARCHAR(256) NOT NULL  -- 非ナル制約,
    email VARCHAR(512) UNIQUE   -- 一意性制約
    membership BOOLEAN,
    register_date DATE,
    company_id INTEGER,
    dpt_id INTEGER REFERENCES Department(id) -- 参照制約1
        ON DELETE CASCADE ON UPDATE RESTRICT,

    -- 参照制約2
    FOREIGN KEY (company_id) REFERENCES Company(id)
        ON UPDATE CASCADE -- オプション(NO ACTION, SET NULLなど)
)
```

<div style="page-break-before:always"></div>

トリガーについてはテーブルのあるデータ更新の際に、別のデータを連動して変更するように設定する処理であり、以下の構文に従う。

```sql
-- 構文
CREATE TRIGGER <トリガー名> <トリガー動作時期> <トリガー事象> 
ON <テーブル名>
    REFERENCING <遷移表 or 遷移変数リスト>
    <被トリガー動作>
```

- <**トリガー動作時期**>
データの更新時期を設定する。データの更新前にトリガーを実行する場合は<font color=red>$BEFORE$</font>、更新後の場合は<font color=red>$AFTER$</font>を使用する。
- <**トリガー事象**>
トリガー対象となる操作を設定する。$INSERT$、$DELETE$、$UPDATE$を指定できる。
- <**REFERENCINGと遷移表**>
値を遷移させる表や変数を指定する。$OLD$、$NEW$で新旧を指定し、$ROW$、$TABLE$で遷移対象を指定する。

```sql
-- 具体例
CREATE TRIGGER TR1 AFTER UPDATE OF 引当済数量 
ON 在庫
    REFERENCING NEW ROW AS CUR_ROW FOR EACH ROW
    WHEN (CUR_ROW.実在庫数量 - CUR_ROW.引当済数量 <= CUR_ROW.基準在庫数量)
    BEGIN ATOMIC
        -- 発注を行うプロシージャの呼出
        CALL PARTSORDER(CUR_ROW.部品番号); 
    END
```

### SQL-DCL(Data Control Language)

```sql
-- 【構文】アクセス権限
GRANT { ALL [PRIVILEGES] } | SELECT | INSERT | DELETE | UPDATE | REFERENCES | USAGE 
ON {[TABLE] 表(またはビュー)} | DOMAIN ドメイン名
TO {ユーザ名 | PUBLIC } [WITH GRANT OPTION]

-- 具体例
GRANT ALL PRIVILEGES 
ON 商品 
TO うさぎ
```

```sql
-- トランザクション管理
START TRANSACTION {
    { READ ONLY | READ WRITE } [,...] | 
    ISOLATION LEVEL { -- 分離レベル設定
        READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE
    } |
    DIAGNOSTIC SIZE int(整数)
};
```

### データ分析のためのSQL

#### データ処理・加工のためのSQL

1. **CASE関数** 条件分岐処理の関数。`SELECT`や`UPDATE`、`DELETE`など柔軟な処理が可能。
2. **日付/時刻関数** `CURRENT_DATE`や`CURRENT_TIMESTAMP`などの日付や時刻を表す関数。
3. **COALESCE関数** 欠損値(NULL)をデフォルト値に置き換える関数。
4. **CONCAT関数** 文字列結合の関数。

```sql
-- CASE式
CASE
    WHEN <条件式> THEN <戻り値>
    WHEN <条件式> THEN <戻り値>
    ...
    ELSE <戻り値>
END

-- 日付/時間/日時関数
SELECT 
    CURRENT_DATE,       -- 現在日付 【例】2025-08-09
    CURRENT_TIME,       -- 現在時間 【例】10:48:41.105768+09
    CURRENT_TIMESTAMP   -- 現在日時 【例】2025-08-09 10:45:05.424812+09
;

-- COALESCE関数
COALESCE(A, B, 0) -- AがNULLであればB、BがNULLであれば0を返す

-- CONCAT関数
SELECT CONCAT('abc', '123'); -- 「abc123」を返す
```

### ウィンドウ関数

ウィンドウ関数は$標準SQL(SQL:2003以降)$から導入された関数であり、<font color=red>`OVER`句を用いて順序や範囲に応じた集計を簡単に行うための関数群</font>である。

```sql
-- 【構文】ウィンドウ関数
<関数> OVER (
    [PARTITION BY 列名]
    [ORDER BY 列名1, 列名2, ...]
    [ROWS/RANGE BETWEEN 開始点 AND 終了点]
)
```

<table>
    <caption>ウィンドウ関数</caption>
	<tbody>
		<tr>
			<th>関数</th>
			<th>説明</th>
		</tr>
		<tr>
			<td>ROW_NUMBER()</td>
			<td>各行に順に一意となる行番号を付与</td>
		</tr>
		<tr>
			<td>RANK()</td>
			<td>ランキング(同率で番号を飛ばした値)を付与</td>
		</tr>
		<tr>
			<td>DENSE_RANK()</td>
			<td>ランキング(同率で番号を飛ばさない値)を付与</td>
		</tr>
		<tr>
			<td><font color=red>LAG(列名[,n])</td>
			<td>n行<b>前</b>の行の値を取得</td>
		</tr>
		<tr>
			<td><font color=red>LEAD(列名[,n])</td>
			<td>n行<b>後</b>の行の値を取得</td>
		</tr>
		<tr>
			<td>AVG(列名)</td>
			<td>ウィンドウ内の該当する列の平均</td>
		</tr>
		<tr>
			<td>SUM(列名)</td>
			<td>ウィンドウ内の該当する列の合計</td>
		</tr>
		<tr>
			<td>MAX(列名)</td>
			<td>ウィンドウ内の該当する列の最大値</td>
		</tr>
		<tr>
			<td>MIN(列名)</td>
			<td>ウィンドウ内の該当する列の最小値</td>
		</tr>
		<tr>
			<td>COUNT(列名)</td>
			<td>ウィンドウ内の該当する列のレコード数</td>
		</tr>
	</tbody>
</table>

<div style="page-break-before:always"></div>

### WITH句

WITH句は<font color=red><b>仮想テーブルを作成する</b>ものであり、共通テーブル式またはCTE(Common Table Expressions)と呼ばれる</font>。

```sql
-- 【構文】WITH句
WITH [RECURSIVE] <仮想テーブル名>(<列名リスト>) AS (
    <問合せ内容>
)

-- 【例】仮想テーブルTEMPのTOTALカラムに、物件テーブルのレコード数を格納する。
WITH TEMP ( TOTAL ) AS (
    SELECT COUNT(*) FROM 物件
)

-- 【例】作成した仮想テーブルTEMPを用いて全物件数に占める割合を百分率で算出
SELECT 
    沿線, 
    FLOOR( COUNT(*) * 100 / TOTAL ) -- 全物件数に占める割合
  FROM 物件 
 CROSS JOIN TEMP
 WHERE エアコン='Y' AND オートロック = 'Y'
 ORDER BY 沿線;
```

<div style="page-break-before:always"></div>

## SQLのポイント

### グループ化



### 結合



### 副問合せ



<div style="page-break-before:always"></div>

## SQLでできること

### ビュー



### カーソル



### ストアドプロシージャ


