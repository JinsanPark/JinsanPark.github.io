---
title: html/css
date: 2026-02-21 16:31:00 +0900
categories: [html, css]
tags: [html, 백엔드]
---

이번에는 제가 그 동안 혼자 배웠던 html및 css 간단하게 정리해두는 글입니다.
백엔드 공부라서, 엄청 자세하게 다루지는 않습니다.

html은 기본적으로 태그를 이용하여 작성해야합니다.

예시
```html
<p>글 본문</p>
<h1>글 제목</h1>
<img src="이미지 경로">
<a href="">링크</a>
<button>버튼</button>
<ul><li>리스트</li></ul>
<ol><li>리스트</li></ol>
```
이런 식으로요.
용도에 맞는 태그를 써야, 나중에 알아보기가 쉽습니다.

링크 아래 처럼 달아주면 됩니다.
```html
<a href="http://naver.com">
  <img src="">
</a>
```

꾸미는건 style을 이용해서 꾸며 줍니다.
```html
<p style="font-size : 20px; color : red"> 글자 </p>
<p><span style="color : red;">글</span style="color : blue;">자</p>
```
아래 코드 처럼 각각도 적용이 가능 합니다.

그런데 저렇게 작성하면 너무 지저분해져서, 따로 CSS라는 파일을 만들어서 연동해줍니다.
```html
 <link href="../css/main.css" rel="stylesheet">
```
이런식 입니다.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link href="../css/main.css" rel="stylesheet">
</head>
<body>
    
    <div class = "css-test">

    </div>

</body>
</html>
```
```css
.css-test { 
  width : 200px;
  display : block;
  margin : auto;
}
```
이렇게 씁니다.

```css
.profile { font-size : 20px }  /*클래스*/
#special { font-size : 30px } /*아이디*/
p { font-size : 16px } /*태그*/
```
이런 식으로도 사용 가능한데,