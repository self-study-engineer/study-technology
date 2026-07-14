<style>
    body {
      counter-reset: chapter 1;
    }
    h1 {
        counter-reset: sub-chapter 5;
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

# 要求仕様を書く

@import "chap2/sec6.md"

<div style="page-break-before:always"></div>

@import "chap2/sec7.md"

<div style="page-break-before:always"></div>

@import "chap2/sec8.md"

<div style="page-break-before:always"></div>

@import "chap2/sec9.md"

<div style="page-break-before:always"></div>

@import "chap2/sec10.md"

<div style="page-break-before:always"></div>

@import "chap2/sec11.md"

<div style="page-break-before:always"></div>

@import "chap2/sec12.md"
