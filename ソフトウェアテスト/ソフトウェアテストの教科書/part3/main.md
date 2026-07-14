<style>
    body {
      counter-reset: chapter 2;
    }
    h1 {
        counter-reset: sub-chapter 8;
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

# テストドキュメントとモニタリング

- 本章では、①テストドキュメントの役割、②各ドキュメントに含まれる項目、③テスト時のプロジェクトチームの構成と各メンバーの役割、の3つの内容を解説する。

@import "chap9.md"

<div style="page-break-before:always"></div>

@import "chap10.md"

<div style="page-break-before:always"></div>

@import "chap11.md"