<style>
    body {
      counter-reset: chapter;
    }
    h1 {
        counter-reset: sub-chapter;
    }
    h2 {
        counter-reset: section;
    }

    h1::before {
        counter-increment: chapter;
    }
    h2::before {
        counter-increment: sub-chapter;
        content: counter(sub-chapter) "章. ";
    }
    h3::before {
        counter-increment: section;
        content: counter(sub-chapter) "-" counter(section) ". ";
    }
</style>

# 達人に学ぶDB設計徹底指南書

@import "chap1.md"

<div style="page-break-before:always"></div>

@import "chap2.md"

<div style="page-break-before:always"></div>

@import "chap3.md"

<div style="page-break-before:always"></div>

@import "chap4.md"

<div style="page-break-before:always"></div>

@import "chap5.md"

<div style="page-break-before:always"></div>

@import "chap6.md"
 
<div style="page-break-before:always"></div>

@import "chap7.md"

<div style="page-break-before:always"></div>

@import "chap8.md"

<div style="page-break-before:always"></div>

@import "chap9.md"
