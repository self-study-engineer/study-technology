<style>
    body {
      counter-reset: chapter 1;
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

# RDBの世界

@import "chap2/sec13.md"

<div style="page-break-before:always"></div>

@import "chap2/sec14.md"

<div style="page-break-before:always"></div>

@import "chap2/sec15.md"

<div style="page-break-before:always"></div>

@import "chap2/sec16.md"

<div style="page-break-before:always"></div>

@import "chap2/sec17.md"

<div style="page-break-before:always"></div>

@import "chap2/sec18.md"

<div style="page-break-before:always"></div>

@import "chap2/sec19.md"

<div style="page-break-before:always"></div>

@import "chap2/sec20.md"

<div style="page-break-before:always"></div>

@import "chap2/sec21.md"

<div style="page-break-before:always"></div>

@import "chap2/sec22.md"

<div style="page-break-before:always"></div>

@import "chap2/sec23.md"