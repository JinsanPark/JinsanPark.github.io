---
title: html/css
date: 2026-02-21 16:31:00 +0900
categories: [html, css]
tags: [html, 백엔드]
---

JS로 UI만드는 법
html/css로 미리 디자인 한 다음,
필요할 때 자바스크립트를 통해 보여주도록 한다.

 jQuery 사용시 코드 단축가능
 ```html
    <script>

        document.querySelector('.navbar-toggler').addEventListener("click", function () {
            document.querySelectorAll('.list-group')[0].classList.toggle('show')
        })
        
    </script>
 ```

  ```html
    <script>

        $('.navbar-toggler').on("click", function(){
            $('.list-group').first().toggleClass('show')
        })

    </script>
 ```
 처럼 줄일 수 있다.