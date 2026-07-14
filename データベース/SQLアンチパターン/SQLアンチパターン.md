<style>
    body {
      counter-reset: chapter;
      counter-reset: section;
    }
    h1 {
        counter-reset: sub-chapter;
    }

    h1::before {
        counter-increment: chapter;
    }
    h2::before {
        counter-increment: sub-chapter;
        content: "第" counter(sub-chapter) "部 ";
    }
    h3::before {
        counter-increment: section;
        content: counter(section) "章 ";
    }
</style>

# SQLアンチパターン

### アンチパターンとは何か？

<div style="page-break-before:always"></div>

## データベース論理設計のアンチパターン

@import "chap1/sec2.md"

<div style="page-break-before:always"></div>

@import "chap1/sec3.md"

<div style="page-break-before:always"></div>

@import "chap1/sec4.md"

<div style="page-break-before:always"></div>

@import "chap1/sec5.md"

<div style="page-break-before:always"></div>

@import "chap1/sec6.md"

<div style="page-break-before:always"></div>

@import "chap1/sec7.md"

<div style="page-break-before:always"></div>

@import "chap1/sec8.md"

<div style="page-break-before:always"></div>

@import "chap1/sec9.md"

<div style="page-break-before:always"></div>

## データベース物理設計のアンチパターン

@import "chap2/sec10.md"

<div style="page-break-before:always"></div>

@import "chap2/sec11.md"

<div style="page-break-before:always"></div>

@import "chap2/sec12.md"

<div style="page-break-before:always"></div>

@import "chap2/sec13.md"

<div style="page-break-before:always"></div>

## クエリのアンチパターン

@import "chap3/sec14.md"

<div style="page-break-before:always"></div>

@import "chap3/sec15.md"

<div style="page-break-before:always"></div>

@import "chap3/sec16.md"

<div style="page-break-before:always"></div>

@import "chap3/sec17.md"

<div style="page-break-before:always"></div>

@import "chap3/sec18.md"

<div style="page-break-before:always"></div>

@import "chap3/sec19.md"

<div style="page-break-before:always"></div>

## アプリケーション開発のアンチパターン

@import "chap4/sec20.md"

<div style="page-break-before:always"></div>

@import "chap4/sec21.md"

<div style="page-break-before:always"></div>

@import "chap4/sec22.md"

<div style="page-break-before:always"></div>

@import "chap4/sec23.md"

<div style="page-break-before:always"></div>

@import "chap4/sec24.md"

<div style="page-break-before:always"></div>

@import "chap4/sec25.md"

<div style="page-break-before:always"></div>

## 外部キーのミニ・アンチパターン

@import "chap5/sec26.md"

<div style="page-break-before:always"></div>

@import "chap5/sec27.md"
