<style>
    body {
      counter-reset: chapter 2;
    }
    h1 {
        counter-reset: sub-chapter 12;
    }
    h2 {
        counter-reset: section;
    }

    h1::before {
        counter-increment: chapter;
        content: counter(chapter) "部 ";
    }
    h2::before {
        counter-increment: sub-chapter;
        content: counter(sub-chapter) "章 ";
    }
    h3::before {
        counter-increment: section;
    }
</style>

# 要件管理と計測

@import "chap3/sec13.md"

<div style="page-break-before:always"></div>

@import "chap3/sec14.md"

<div style="page-break-before:always"></div>

@import "chap3/sec15.md"
