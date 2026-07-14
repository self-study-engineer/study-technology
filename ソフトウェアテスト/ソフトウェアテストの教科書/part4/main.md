<style>
    body {
      counter-reset: chapter 3;
    }
    h1 {
        counter-reset: sub-chapter 11;
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

# 次のステップへ

- Part1～3を活かしながら、本章では以下の内容を解説する。
  - ①アジャイル開発
  - ②テスト駆動開発
  - ③テスト自動化

@import "chap12.md"

<div style="page-break-before:always"></div>

@import "chap13.md"
