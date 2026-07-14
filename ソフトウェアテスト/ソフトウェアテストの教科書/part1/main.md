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

# ソフトウェアテストの基本

- 本章では以下の内容を解説する。
  - ①ソフトウェアテストの概要
  - ②社会におけるソフトウェアテストの役割
  - ③開発に携わるすべてのエンジニアに必要な品質意識
  - ④ソフトウェア開発におけるソフトウェアテストの位置づけ

@import "chap1.md"

<div style="page-break-before:always"></div>

@import "chap2.md"

<div style="page-break-before:always"></div>

@import "chap3.md"
