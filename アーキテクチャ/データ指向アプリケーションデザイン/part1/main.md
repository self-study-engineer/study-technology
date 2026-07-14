<style>
    body {
      counter-reset: chapter 0;
    }
    h1 {
        counter-reset: sub-chapter 0;
    }
    h2 {
        counter-reset: section;
    }
    h3 {
        counter-reset: sub-section;
    }

    h1::before {
        counter-increment: chapter;
        content: "第" counter(chapter) "部 ";
    }
    h2::before {
        counter-increment: sub-chapter;
        content: counter(sub-chapter) "章 ";
    }
    h3::before {
        counter-increment: section;
        content: counter(sub-chapter) "." counter(section) " ";
    }
    h4::before {
        counter-increment: sub-section;
        content: counter(sub-chapter) "." counter(section) "." counter(sub-section) " ";
    }
    ol.brackets li {
        font-family: 'Times New Roman', serif;
        list-style-type: none;
        counter-increment: cnt;
    }
    ol.brackets li:before {
        content: "[ " counter(cnt) " ]　";
    }
</style>

# データシステムの基礎

- 最初の4つの章では、単一のマシンで動作するものから複数のマシンで構成されるクラス上で分散しているものまで、あらゆるデータシステムに当てはまる基本的な概念を見ていく。<font color=red>続く第2部では分散データシステム固有の問題に目を向ける</font>。
- 1章では、本書全体を通して使うことになる用語とアプローチを紹介する。この章では、**信頼性、スケーラビリティ、メンテナンス性**といった言葉が何を意味するのか、そしてそれらの目標をどのように達成すれば良いのかを見ていく。
- 2章では、いくつかのデータモデルとクエリ言語を比較する。開発者の視点から見ればこれらがDB間で最も目に見える歳になり、様々な状況に対して様々なモデルが適している様子を見ていく。
- 3章では、ストレージエンジンの内部に目を向け、DBのディスク上のデータのレイアウトを見ていく。様々なストレージエンジンが様々なワークロードに対して最適化されており、適切なエンジンを選択することはパフォーマンスに大きな影響を及ぼす。
- 4章では、データのエンコーディング（シリアライゼーション）のための様々なフォーマットを比較する。特に、時間と共にアプリケーションの要件が変化し、スキーマがその変化に対応しなければならないような環境に、それらのフォーマットがどのように対応するのかを調べる。

<div style="page-break-before:always"></div>

@import "chap1.md"

<div style="page-break-before:always"></div>

@import "chap2.md"

<div style="page-break-before:always"></div>

@import "chap3.md"

<div style="page-break-before:always"></div>

@import "chap4.md"
