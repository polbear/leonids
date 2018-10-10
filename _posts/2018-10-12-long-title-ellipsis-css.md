---
layout: post
title: 게시판에서 긴제목 CSS로 자동으로 줄이기
#excerpt: "excerpt"
categories: [Front-End]
tags: [css,ellipsis]
comments: true
---

대부분의 웹개발자가 거쳐야 하는 과정중에 하나는 바로 게시판을 만드는 것이다.
게시판을 만들다보면, 게시판 리스트에서 출력될 텍스트(흔히 TITLE)의 길가 짧은 것도 긴 것도 존재하기 마련이다.
그때 어떤 행은 개행이되고, 어떤행은 안돼서 모양이 이쁘지 않을 때가 있다.

이런경우 CSS를 이용하여 특정 문자열 길이 이후부터는 점(...)처리를 할 수 있다.

리스트 출력시, 출력하는 텍스트가 너무 길 경우 자동으로 점(...) 처리하도록 CSS 설정으로 해보자.
<!-- more -->

```css
.shorttitle {
    width: 380px;                /* 가로 길이 고정*/
    text-overflow: ellipsis;     /* 생략 처리 ( ... )*/
    white-space: nowrap;         /* 줄바꿈 하지 않고 잘림*/
    overflow: hidden;            /* 스크롤 처리 하지 않음*/
}
```
