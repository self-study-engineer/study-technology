## 3値論理とNULL

<div style="padding: 10px; margin-bottom: 10px; border: 5px double;">
    <h5>学習のポイント</h5>
    <ul>
        <li>NULLは値ではない。そのため述語もまともに適用できない。</li>
        <li>無理やり適用すると<b>unknown(NULL)</b>が生じる。</li>
        <li><font color=red><b>unknown</b>が論理演算に紛れ込むとSQLが直感に反する動作をするため、デバッグは段階的にSQLを分けてステップを追うしかない。</font></li>
        <li>NULLがもたらす問題の対処法はNOT NULL制約をつけて極力NULLを排除することである。</li>
    </ul>
</div>

#### 本題に入る前に

SQLの真理値型は**true**、**false**、**unknown**の3つの値を持ち、普通のプログラミング言語の真理値型とは異なる。そのため**SQLの論理体系は3値論理と呼ばれる**。unknownが発生する理由はRDBにNULLが存在するためあり、これがトリッキーで直感に反する振る舞いを見せる。以降、理論編と実践編に分けて話を進める。

<div style="page-break-before:always"></div>

#### 理論編

##### 2つのNULL、3値論理、それとも4値論理？

NULLは「**未知(Unknown)**」と「**適用不能(Not Applicable, Inapplicable)**」の2種類に区別ができる。

- **未知(Unknown)** 調べてみないことには「わからない」というニュアンス。<font color=red>今はわからないが条件がそろえばわかる</font>。
→ サングラスをかけた人の目の色
- **適用不能(Not Applicable, Inapplicable)** 無意味、論理的に不可能、どう頑張ってもわからない。N/Aと同じ意味。
→ 冷蔵庫の目の色、円の堆積、男性の出産回数など

```plantuml
title RDBにおける失われた情報の分類(E.F.コッド)

storage 失われた情報 as losted_info
storage タプルまるごと as tuple
storage 属性の値 as attributes
storage 不成立の事態 as un_success_event
storage 成立し得ない事態 as inapplicable_event
storage "成立したけどDBMSは\nそのことをしれない事態" as unknown_event
storage "<color red>**未知**\n<color red>Unknown" as unknown
storage "<color red>**適用不能**\n<color red>Not Applicable" as not_applicable
storage その他 as other

losted_info -- tuple
losted_info -- attributes
tuple -- un_success_event
tuple --- inapplicable_event
tuple -- unknown_event
attributes -- unknown
attributes --- not_applicable
attributes -- other
```

<div style="page-break-before:always"></div>

##### なぜ「=NULL」ではなく「IS NULL」なのか？

「NULLは値でも変数でもない」ため、比較演算子を適用した結果が常に**unknown**になる。つまり、NULLは「そこに値がない」ことを示すただの視覚的マーク、目印にすぎない。

```sql
-- 以下の式は全部unknown(≠Unknown)に評価される
1 = NULL
2 > NULL
3 < NULL
4 <> NULL
NULL = NULL
NULL > NULL
NULL < NULL
NULL <> NULL
```

##### unknown、第三の真理値

unknownは真理値のためunknownの比較は可能。しかし、UNKNOWNはNULLの一種のため比較は不可。

```sql
-- これはれっきとした真理値の比較
unknown = unknown -- true

-- UNKNOWNはNULLの一種であるため、要するに「NULL = NULL」
UNKNOWN = UNKNOWN -- unknown
```

また、ANDやORを用いた3値論理について以下に表を示す。加えて、ANDとORにおける3つの真理値の優先順位は以下の通り。

<table>
    <tr>
        <td>
            <table>
                <tbody>
                    <tr>
                        <th>x</th>
                        <th>NOT x</th>
                    </tr>
                    <tr>
                        <td>t</td>
                        <td>f</td>
                    </tr>
                    <tr>
                        <td>u</td>
                        <td>u</td>
                    </tr>
                    <tr>
                        <td>f</td>
                        <td>t</td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
            <table>
                <tbody>
                    <tr>
                        <th>AND</th>
                        <th>t</th>
                        <th>u</th>
                        <th>f</th>
                    </tr>
                    <tr>
                        <th>t</th>
                        <td><font color=red>t</td>
                        <td><font color=green>u</td>
                        <td><font color=blue>f</td>
                    </tr>
                    <tr>
                        <th>u</th>
                        <td><font color=green>u</td>
                        <td><font color=green>u</td>
                        <td><font color=blue>f</td>
                    </tr>
                    <tr>
                        <th>f</th>
                        <td><font color=blue>f</td>
                        <td><font color=blue>f</td>
                        <td><font color=blue>f</td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
            <table>
                <tbody>
                    <tr>
                        <th>OR</th>
                        <th>t</th>
                        <th>u</th>
                        <th>f</th>
                    </tr>
                    <tr>
                        <th>t</th>
                        <td><font color=red>t</td>
                        <td><font color=red>t</td>
                        <td><font color=red>t</td>
                    </tr>
                    <tr>
                        <th>u</th>
                        <td><font color=red>t</td>
                        <td><font color=green>u</td>
                        <td><font color=green>u</td>
                    </tr>
                    <tr>
                        <th>f</th>
                        <td><font color=red>t</td>
                        <td><font color=green>u</td>
                        <td><font color=blue>f</td>
                    </tr>
                </tbody>
            </table>
        </td>
    </tr>
</table>

- 【**ANDの場合**】$false > unknown > true$
- 【**ORの場合**】$true > unknown > false$

<div style="page-break-before:always"></div>

#### 実践編

##### 比較述語とNULL

「宇宙人はいるか、いないかのどちらかである」や「ジョンは20歳か、20歳ではないかのどちらかである」のように<font color=red>「命題とその否定をまたはでつなげてできる命題は全て真である」という命題を2値論理で<b>排中律(excluded middle)</b>と呼びます。</font>
　例えば、以下の生徒テーブルを考える。ここでジョンの年齢はNULLである。

<table>
    <caption>生徒</caption>
    <thead>
    <tr>
        <th>氏名</th>
        <th>年齢</th>
    </tr>
    </thead>
    <tbody>
        <tr><td>ブラウン</td><td>22</td></tr>
        <tr><td>ラリー</td><td>19</td></tr>
        <tr><td>ジョン</td><td></td></tr>
        <tr><td>ボギー</td><td>21</td></tr>
    </tbody>
</table>

```sql
-- このクエリは生徒テーブルの全レコードが出力される。
SELECT * FROM Students

-- このクエリを実行した場合「ジョン」は出力されない。
SELECT * FROM Students
WHERE age = 20 OR age <> 20;
```

以下のようにCASE式でNULLを使用する場合、「IS NULL」を使わなければならない。

```sql
-- 【正しくない記述】
-- ageがNULLでも「IS NOT NULL」で処理されてしまう。
SELECT *,
	CASE age WHEN NULL THEN 'IS NULL' -- age = NULL と認識されるからダメ
	ELSE 'IS NOT NULL' END AS ISNULL
FROM Students;

-- 【正しい記述】
SELECT *,
	CASE WHEN age IS NULL THEN 'IS NULL'
	ELSE 'IS NOT NULL' END AS ISNULL
FROM Students;
```

##### NOT INとNOT EXISTSは同値ではない

INをEXISTSで書き換えることはパフォーマンスチューニングのテクニックとしてよく行われる。しかし、NOT INをNOT EXISTSで書き換える場合、必ずしも結果が一致しないケースがある。つまり、<font color=red><b>INとEXISTSは同値変換可能であるが、NOT INとNOT EXISTSは同値ではない</b></font>。INは「その中に」であり、EXISTSは「少なくとも一つ」であることが理由である。

<table>
    <tr>
        <td>
            <table>
                <caption>ClassA</caption>
                <tbody>
                    <tr>
                        <th>name</th>
                        <th>age</th>
                        <th>city</th>
                    </tr>
                    <tr>
                        <td>ブラウン</td>
                        <td>22</td>
                        <td>東京</td>
                    </tr>
                    <tr>
                        <td>ラリー</td>
                        <td>19</td>
                        <td>埼玉</td>
                    </tr>
                    <tr>
                        <td>ボギー</td>
                        <td>21</td>
                        <td>千葉</td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
            <table>
                <caption>ClassB</caption>
                <tbody>
                    <tr>
                        <th>name</th>
                        <th>age</th>
                        <th>city</th>
                    </tr>
                    <tr>
                        <td>斉藤</td>
                        <td>22</td>
                        <td>東京</td>
                    </tr>
                    <tr>
                        <td>田尻</td>
                        <td>23</td>
                        <td>東京</td>
                    </tr>
                    <tr>
                        <td>山田</td>
                        <td></td>
                        <td>東京</td>
                    </tr>
                    <tr>
                        <td>和泉</td>
                        <td>18</td>
                        <td>千葉</td>
                    </tr>
                    <tr>
                        <td>武田</td>
                        <td>20</td>
                        <td>千葉</td>
                    </tr>
                    <tr>
                        <td>石川</td>
                        <td>19</td>
                        <td>神奈川</td>
                    </tr>
                </tbody>
            </table>
        </td>
    </tr>
</table>

上記のテーブルを例に以下の**NOT IN**のクエリを段階的にみていく。

```sql
-- Bクラスの東京在住の生徒と年齢が一致しないAクラスの生徒を選択するSQL
SELECT * FROM ClassA
WHERE age NOT IN (SELECT age FROM ClassB WHERE city = '東京');

-- ##### 以下、段階的実行結果 #####
-- 1. サブクエリを実行して年齢リストを取得
SELECT * FROM ClassA WHERE age NOT IN (22, 23, NULL);
-- 2. NOT IN を NOT と IN を使って同値変換
SELECT * FROM ClassA WHERE NOT age IN (22, 23, NULL);
-- 3. IN 述語を ORで同値変換
SELECT * FROM ClassA WHERE NOT (age = 22 OR age = 23 OR age = NULL);
-- 4. ド・モアブルの定理適用
SELECT * FROM ClassA WHERE (age <> 22) AND (age <> 23) AND (age <> NULL);
-- 5. NULLに比較演算子は使えない。
SELECT * FROM ClassA WHERE (age <> 22) AND (age <> 23) AND unknown;
-- 6. trueにならない。
SELECT * FROM ClassA WHERE unknown OR false
```

次に、**NOT EXISTS**を見る。

```sql
SELECT * FROM ClassA A
WHERE NOT EXISTS(SELECT * FROM ClassB B WHERE A.age = B.age AND B.city = '東京');

-- ##### 以下、段階的実行結果 #####
-- 1. サブクエリにおいてNULLとの比較を行う。
SELECT * FROM ClassA A
WHERE NOT EXISTS(SELECT * FROM ClassB B WHERE A.age = NULL AND B.city = '東京');
-- 2. NULLに=を適用するとunknownになる。
SELECT * FROM ClassA A
WHERE NOT EXISTS(SELECT * FROM ClassB B WHERE unknown AND B.city = '東京');
-- 3. EXISTS句の中がtrueにならない。
SELECT * FROM ClassA A
WHERE NOT EXISTS(SELECT * FROM ClassB B WHERE unknown OR false);
-- 4. サブクエリの中が結果を返さないのでNOT EXISTS句はtrueになる。
SELECT * FROM ClassA A WHERE true;
```

##### 極値関数とNULL

「**①極値関数は集計の際にNULLを排除する**」という特徴を持つ一方、「**②極値関数は入力が空テーブルだった場合はNULLを返す**」という仕様も持つ。そのため、以下のクエリにおいて①であれば値を返すが、②の場合は$COALESCE$関数を使ってNULLを適当な値に変換すれば対応可能であるが、特に何もしなければ1行も返さないという事象が発生しうる。しかし、<u><font color=red><b>全行を返す</b>のと<b>1行も返さない</b>のと、どちらが望ましいかは要件によるだろう</font></u>。

```sql
SElECT * FROM ClassA
WHERE age < (
    SELECT MIN(age) FROM ClassB
    WHERE city = '東京'
)
```

##### 集約関数とNULL

**COUNT関数以外の集約関数(SUMやAVGなど)も入力が空テーブルだった場合はNULLを返す**。<font color=red>集約関数(COUNT以外)と極値関数について、入力が空のテーブルだった場合はNULLを返すという仕様について認識しておかなければ不具合の原因になるため注意が必要</font>。
