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
- DOCTYPE 是 document type 的缩写，DOCTYPE 声明位于 html 文档的最前面，用于告知浏览器用哪种规范解析该文档。`<!DOCTYPE html>` 即声明本文档为 HTML5 文档。

### 二、html 标签 - 根元素
```html
<!DOCTYPE html>
<html>
</html>
```
- 此标签标识其为 HTML 文档，是文档的根元素。DOCTYPE 必须位于 `<html>` 标签之前

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
- `<meta>` 标签提供了 HTML 文档的元数据。
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
- `<title>` 标签定义了文档的标题，通常显示在浏览器的标题栏或状态栏上。

例图：
![title](/assets/images/html_examples/title-example.png)

#### 3. base 标签
```html
<!DOCTYPE html>
<html>
<!--- 头部  --->
<head>
    <meta name="author" content="beirsnow">
    <meta charset="UTF-8">
    <title>beirsnow-html</title>
    <base href="https://www.beirsnow.com/" target="_blank" />
</head>
<!--- 主体 --->
<body>
    <a href="archives.html">archives page</a>
</body>
</html>
```
- `<base>` 为页面上所有的链接指定基地址。
如上述示例中，指定 base 为 "https://www.beirsnow.com/"，则在 body 中的相对URL "archives.html" 会被浏览器自动填充为 "https://www.beirsnow.com/archives.html"
- `<base>` 可选属性 **target** 可指定在何处打开页面中的链接。  
_blank: 在新窗口中打开被链接文档  
_parent：在父框架集中打开被链接文档  
_self：默认，在相同的框架中打开被链接文档  
_top：在整个窗口中打开被链接文档  
framename: 在指定的框架中打开被链接文档
- 示例中出现的 `<!--- 头部 --->` 为 HTML 的注释，以 `<!---` 开始，`--->` 结束。
**注意：base 规定的是页面上所有链接的默认 URL 基址，本页面所有链接路径必须在 base 标签内，不然将无法找到**

#### 4. link 标签
```html
<!DOCTYPE html>
<html>
<!--- 头部  --->
<head>
    <meta name="author" content="beirsnow">
    <meta charset="UTF-8">
    <title>beirsnow-html</title>
    <base href="../HTML/" target="_blank" />
    <link rel="stylesheet" type="text/css" href="test.css" />
</head>

<!--- 主体 --->
<body>
    <h1>test heading</h1>
</body>
</html>
```
- `<link>` 标签定义文档与外部资源的关系, 主要用于链接样式表。  
- `<link>` 常用属性如下：  
rel：规定当前文档与被链接文档之间的关系  
type：规定被链接文档的 MIME 类型  
href：规定被链接文档的位置

示例中 `<link>` 标签即为此文档定义了层叠样式表(CSS)
层叠样式表 "test.css" 中的内容如下：
```css
h1{color:red;}
```
示例效果如下：
![link-css](/assets/images/html_examples/link-example.png)

**未完待续...**