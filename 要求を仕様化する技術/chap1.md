<style>
    body {
      counter-reset: chapter 0;
    }
    h1 {
        counter-reset: sub-chapter;
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

# 要求仕様にまつわる問題

@import "chap1/sec1.md"

<div style="page-break-before:always"></div>

@import "chap1/sec2.md"

<div style="page-break-before:always"></div>

@import "chap1/sec3.md"

<div style="page-break-before:always"></div>

@import "chap1/sec4.md"

<div style="page-break-before:always"></div>

@import "chap1/sec5.md"
