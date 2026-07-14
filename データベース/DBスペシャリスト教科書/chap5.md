<style>
    body {
      counter-reset: chapter 4;
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

# DBMS

## DBMSとは




<div style="page-break-before:always"></div>

## トランザクション管理




<div style="page-break-before:always"></div>

## 障害回復処理




<div style="page-break-before:always"></div>

## 分散データベース



<div style="page-break-before:always"></div>

## 問題演習


