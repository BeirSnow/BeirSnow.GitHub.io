---
layout: post
title:  "呵特么了（HTML）基础学习"
date:   2022-05-17 20:30:00 +0800
categories: HTML
tags: html
---

### 一、DOCTYPE
```html
<!DOCTYPE html>
```
DOCTYPE 是 document type 的缩写,DOCTYPE 声明位于 html 文档的最前面,用于告知浏览器用哪种规范解析该文档。`<!DOCTYPE html>` 即声明本文档为 HTML5 文档。

### 二、html 标签 - 根元素
```html
<!DOCTYPE html>
<html>
</html>
```
此标签标识其为 HTML 文档，是文档的根元素。DOCTYPE 必须位于 `<html>` 标签之前
>`<html>` 与 `</html>` 标签限定了文档的开始点和结束点 --- [w3school](https://www.w3school.com.cn)

### 三、head 标签 - 头部
```html
<!DOCTYPE html>
<html>
<head>
</head>    
</html>
```
>`<head>` 标签用于定义文档的头部，它是所有头部元素的容器。`<head>` 中的元素可以引用脚本、指示浏览器在哪里找到样式表、提供元信息等等。
文档的头部描述了文档的各种属性和信息，包括文档的标题、在 Web 中的位置以及和其他文档的关系等。绝大多数文档头部包含的数据都不会真正作为内容显示给读者。 --- [w3school](https://www.w3school.com.cn)

#### 1. meta 标签
```html
<!DOCTYPE html>
<html>
<head>
<meta name="author" content="beirsnow">
<meta charset="UTF-8">
</head>    
</html>
```
`<meta>` 标签提供了 HTML 文档的元数据。
- `<meta>` 标签通常位于 `<head>` 标签内
- 元数据(meta)通常以 名称/值 对出现
- 如果没有提供 name 属性，那么名称/值对中的名称会采用 http-equiv 属性的值（使用带有 http-equiv 属性的 `<meta>` 标签时，服务器将把名称/值对添加到发送给浏览器的内容头部）

#### 2. title 标签
```html
<!DOCTYPE html>
<html>
<head>
<meta name="author" content="beirsnow">
<meta charset="UTF-8">
<title>beirsnow-html</title>
</head>    
</html>
```
`<title>` 标签定义了文档的标题，通常显示在浏览器的标题栏或状态栏上。

例图：
![title](/assets/images/html_examples/title-example.png)

**未完待续...**