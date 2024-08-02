---
title: "Markdown PDF 변환 시 Latex Math 수식 깨짐 현상 해결 방법"
excerpt: "Markdown PDF 변환 시 Latex Math 수식 깨짐 현상 해결 방법"

categories:
    - Blog
tags:
    - Blog
last_modified_at: 2023-06-29T16:52:00
---

마크다운 파일을 PDF로 손쉽게 변환해주는 VSCode 플러그인을 사용하여<br>
포스팅 파일을 변환해보니 수식이 깨져버리는 것을 확인하였고<br>

이를 손쉽게 해결할 수 있는 방법을 찾아 공유해보고자 합니다<br>

# 해결 방법
---
**`Markdown PDF 플러그인의 template.html 파일을 수정`**해야 하는데<br>

해당 파일은 윈도우 기준으로<br>
**`C:\Users\사용자명\.vscode\extensions\yzane.mardown-pdf-다운로드 버전\template\template.html`**
위 경로에 존재합니다 (기본 설치 경로를 따로 변경하지 않으셨다면)<br>

해당 파일을 열어서 아래의 두 줄 코드를 해당 파일에 붙여넣기만 하면<br>
수식이 깨지는 현상을 손쉽게 해결하실 수 있습니다<br>
```html
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config"> MathJax.Hub.Config({ tex2jax: {inlineMath: [['$', '$']]}, messageStyle: "none" });</script>
```

다음과 같이 저는 \<head\>와 \<title\> 사이에 해당 코드를 붙여넣었습니다
```html
<!DOCTYPE html>
<html>
<head>
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config"> MathJax.Hub.Config({ tex2jax: {inlineMath: [['$', '$']]}, messageStyle: "none" });</script>
<title>{{{title}}}</title>
<meta http-equiv="Content-type" content="text/html;charset=UTF-8">
```