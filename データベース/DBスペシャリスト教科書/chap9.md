<style>
    body {
      counter-reset: chapter 8;
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

# 最新データベース技術

## 様々なデータベース



<div style="page-break-before:always"></div>

## データベース周辺技術


<div style="page-break-before:always"></div>

## 問題演習


