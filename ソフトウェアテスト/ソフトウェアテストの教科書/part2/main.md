<style>
    body {
      counter-reset: chapter 1;
    }
    h1 {
        counter-reset: sub-chapter 3;
    }
    h2 {
        counter-reset: section;
    }

    h1::before {
        counter-increment: chapter;
        content: "Part " counter(chapter) " ";
    }
    h2::before {
        counter-increment: sub-chapter;
        content: "Chapter " counter(sub-chapter) " ";
    }
    h3::before {
    }
</style>

# さまざまなテスト技法

<img src="images/2-1.png" width=65%>

- 本章では具体的なテスト技法について4つを解説する。

@import "chap4.md"

<div style="page-break-before:always"></div>

@import "chap5.md"

<div style="page-break-before:always"></div>

@import "chap6.md"

<div style="page-break-before:always"></div>

@import "chap7.md"

<div style="page-break-before:always"></div>

@import "chap8.md"
