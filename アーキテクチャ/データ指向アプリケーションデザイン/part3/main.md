<style>
    body {
      counter-reset: chapter 2;
    }
    h1 {
        counter-reset: sub-chapter 9;
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

# 導出データ

<font size=5><b>【記録のシステム（SoR：System of Record）と導出データ】</b></font>

- 

<font size=5><b>【各章の概要】</b></font>

- 

<div style="page-break-before:always"></div>

@import "chap10.md"

<div style="page-break-before:always"></div>

@import "chap11.md"

<div style="page-break-before:always"></div>

@import "chap12.md"
