---
layout: post
title:  "10/06 (일) 나의 첫 HTML & CSS 웹 디자인 - 파트2"
date:   2024-10-06 11:00:00 +0900
categories: psyoblade created
---

## 루틴: 2024년 10월 6일 (일)

>     

### 새로 배운 지식

#### Head Markup

* Meta : `<meta charset="UTF-8">`
* CSS Link : `<link rel="stylesheet" href="style.css">`

#### Body Markup

* Table `<table><thead><tbody><tr><td>`
* Unordered List `<ul><li>`
* Ordered List `<ol><li>`
* Description List, Term, Description : `<dl><dt><dd>`
* Paragraph : `<p>`
* Address : `<address><a href="tel:010-1234-5678">010-1234-5678</a>`

#### 웹 사이트 영역 구분

* `<header><nav>`
* `<main><article><section><div>`
* `<aside>`
* `<footer>`



### 인사이트

#### VUE 관점에서 HTML 과 CSS 의 효용성

* HTML 에서 CSS 를 로딩하는 것은 개별 엘리먼트의 속성을 변경하기 위함인데, 하나의 CSS 만 사용할 수 있다면 활용성은 많이 떨어질 것 같다
* 일관된 패턴을 적용한다는 차원에서는 좋지만 결과적으로 그때 그때 다른 포맷을 사용하고 싶은데 그럴 수 없기 때문이다
* 하지만 Vue 관점에서 보았을 때에 CSS 가 컴포넌트 단위로 변경할 수 있다면 얘기가 달라지는데 과거에 이미지를 올려서 버튼을 만들었다고 하면 이제는 CSS 와 HTML 조합으로 하나의 버튼 컴포넌트를 만들 수 있다는 것이고 이는 무거운 이미지 대신 가벼운 컴포넌트를 사용할 수 있다는 말이다
* 변수 바인딩을 통해서 동적으로 컴포넌트를 수정하고 변경할 수 있다는 점이 상당히 모듈화가 가능하기 때문에 효과적인 개발이 가능할 것 같다



### 비주얼 스튜디오 코드

#### Emmet 통한 지정한 코드를 태그로 래핑

* 코드를 선택하고 `Shift+Command+P` 통해서 커맨드 선택 `Emmet Abbreviation` 선택하여 `h1` 태그 입력
  * 혹은 `ul>li` 과 같이 중복된 태그도 입력 가능
* 여러 줄에 개별 태그를 입력하고 싶은 경우라면 `Alt` 혹은 `Option` 을 누른 상태에서 클릭을 하면 커서를 여러개 생성해서 태그 입력
  * 

### 질문과 답변

* 이러한 [Destyle](https://github.com/nicolas-cusan/destyle.css) 같은 것은 일반적으로 잘 활용되는 것인가?
* 폰트를 앱과 같이 배포하고 싶은데 어떻게 하면 되는가?
* 웹사이트 영역 구분은 유용할 것 같은데 실제로도 사용하는 태그들인가?