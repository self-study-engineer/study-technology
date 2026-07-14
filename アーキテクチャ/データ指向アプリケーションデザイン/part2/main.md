<style>
    body {
      counter-reset: chapter 1;
    }
    h1 {
        counter-reset: sub-chapter 4;
    }
    h2 {
        counter-reset: section;
    }
    h3 {
        counter-reset: sub-section;
    }

    h1::before {
        counter-increment: chapter;
        content: "第" counter(chapter) "部 ";
    }
    h2::before {
        counter-increment: sub-chapter;
        content: counter(sub-chapter) "章 ";
    }
    h3::before {
        counter-increment: section;
        content: counter(sub-chapter) "." counter(section) " ";
    }
    h4::before {
        counter-increment: sub-section;
        content: counter(sub-chapter) "." counter(section) "." counter(sub-section) " ";
    }
    ol.brackets li {
        font-family: 'Times New Roman', serif;
        list-style-type: none;
        counter-increment: cnt;
    }
    ol.brackets li:before {
        content: "[ " counter(cnt) " ]　";
    }
</style>

# 分散データ

<font size=5><b>【高負荷に対応するスケーリング】</b></font>

- 

<font size=4><b>シェアードナッシングアーキテクチャ</b></font>

- 

<font size=5><b>【レプリケーションとパーティショニング】</b></font>

- 

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
