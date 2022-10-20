---
layout: single
title: "[Jekyll] Clean blog theme code highlight 추가"
categories: Jekyll
date: 2019-04-03
tags: [jekyll, cleanblog]
---

## Code highlight 문제

처음 Github pages로 블로그를 만들었을 때 minimal-mistake 테마를 사용하다가 너무 단순하고 추후 수정이 어려울 것 같아 Clean blog theme 로 변경을 했는데, code highlighting이 적용되지 않았다.

아래 그림의 빨간 박스 부분이 ```를 이용해 넣은 코드 부분인데 본문과 같은 스타일로 표시되어 구분이 되지 않았다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/jekyll_code_highlight_no.png?raw=true">

그래서 테마 변경하는 방법을 찾아보던 중 highlight 관련 내용을 찾아 적용한 부분을 정리해 본다.

참고했던 포스트와 같은 테마는 아니었지만 도움이 되었다.

---

## Clean Blog theme highlight 적용

Clean blog theme의 경우, 다음 경로의 **\_bootstrap-overrides.scss** 파일에 아래 코드를 추가해주면 된다.

- assets/vendor/startbootstrap-clean-blog/scss/\_bootstrap-overrides.scss

배경색, 글자색, 폰트, padding을 적용했으며 내 블로그에 맞는 스타일로 수정 & 적용하면 된다.

```
pre.highlight {
  background: #343434;
  color: #aab1bf;
  font-family: 'D2 coding', "Consolas", monospace;
  padding: 5px;
}
```

---

## Reference

참고 링크

- [JEKYLL의 CODE BLOCK 테마 예쁘게 만들기](https://eungbean.github.io/2018/08/14/use-Atom's-One-Dark-syntax-theme-with-jekyll/)
