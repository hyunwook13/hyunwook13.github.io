---
layout: post
title: Error in publish
tags: [ci/cd, publish, sass]
---

## Web 배포 도중 오류 해결 과정
1. 처음 발생한 오류 

> Rendering Markup: assets/css/main.scss github-pages 232 |
Error: Invalid CSS after " @if meta": expected "{", was ".function-exist..." on line 72

* Invalid CSS로 인한 에러가 발생.  허나 이상한 점이 72번째 줄이 존재하지 않는 상황
* GPT Said :  @if meta.function-exists(...) → **Jekyll 기본 Sass 빌드 도구(libSass)**에서는 **meta. 네임스페이스를 지원하지 않음**

```
❌
 @if meta.function-exists("some-function") {
  // ...
}
```


```
🆗
@if function-exists(some-function) {
  // ...
}
```

2. 동일한 에러가 발생, 허나 문구를 더 확인해보니 
 /github/workspace/_sass/yat.scss:72에서 발생 중 
-> yat.scss를 들어가 수정 시작 

```
// 기존 코드
@function divide($dividend, $divisor: 1) {
  @if meta.function-exists('div', 'math') { // 에러가 나던 부분 
    @return math.div($dividend, $divisor);
  } @else {
    @return ($dividend / $divisor);
  }
}

```

``` 
// 수정 코드 

@function divide($dividend, $divisor:  1) {
	@return ($dividend / $divisor);
}
```

## 수정 완료!

### 추가적으로 해야할 것
1. css VS scss?
2. sass 문법이 뭘까?