# CSS
**CSS一些属性：**  

text-decoration:underline;//下划线  

text-decoration:overline;//上划线  

text-decoration:line-through;//删除线  

text-decoration:blink;//文字闪烁效果（有兼容性问题）  

text-decoration:none;//没有任何的修饰



**CSS注释：**

![image](images/e6fd8259-67d0-4238-b180-d4bccffe2bf5.jpg)



**CSS样式规则\*\*\*\*：**

![image](images/9475bb0b-29f5-478b-a549-a38fce5dd2cb.jpg)

**如何引用CSS样式\*\*\*\*：**  

![image](images/0f5aa24a-f8c5-4f32-b233-3b7f646d6f1f.jpg)

**1.行内样式（内联样式）**  

![image](images/a41eb01a-1937-4729-b824-27cc517e2776.jpg)

**2.内部样式（嵌入样式）**  

![image](images/11dd3042-cba9-4bc1-9fa7-4d810c7d7f7b.jpg)

**3.外部样式（外联样式）**  

![image](images/b5bd8b20-c939-4164-8357-47b38716c8f0.jpg)

**4.导入式**  

![image](images/98ebc093-d625-4011-9971-e9f64c98938d.jpg)





**例1 行内样式（内联样式）:**  

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Css样式</title>
</head>
<body>
    <h1 style="font-size:20px;font-family:隶书;color: blue;">CSS层叠样式</h1>

</body>
</html>
```
**例2 内部样式（嵌入样式）:**  

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Css样式</title>
    <style type="text/css">
        p{font-size:20px;
          color: blue;
          font-family: "隶书";}
        h1{color: red;
           font-size: 30px;}
    </style>
</head>
<body>
    <h1>CSS层叠样式</h1>
    <p>用于定义HTML内容在浏览器中的显示样式</p>
</body>
</html>
```
**例3 外部样式（外联样式）:**  

    **html文件**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Css样式</title>
    <link rel="stylesheet" type="text/css" href="css.css">
</head>
<body>
    <h1>CSS层叠样式</h1>
    <p>用于定义HTML内容在浏览器中的显示样式</p>
</body>
</html>
```
    **css文件**

```html
p{font-size:20px;
  color: blue;
  font-family: "隶书";}

h1{color: red;
  font-size: 30px;}
```
**例4 导入式:**  

    **html文件**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Css样式</title>
    <style type="text/css">
        @import url(css.css);
    </style>
</head>
<body>
    <h1>CSS层叠样式</h1>
    <p>用于定义HTML内容在浏览器中的显示样式</p>
</body>
</html>
```
    **css文件**  

```html
p{font-size:20px;
  color: blue;
  font-family: "隶书";}

h1{color: red;
  font-size: 30px;}
```
**CSS使用方法区别\*\*\*\*：**  

![image](images/b9f42401-7641-4a76-8db2-49eb71688d93.jpg)

![image](images/1ec16685-ac62-46a0-a30b-0b0777f07cb1.jpg)

**CSS样式优先级\*\*\*\*：**  

![image](images/58b09149-96bb-4464-9216-136b2487b3fe.jpg)

**CSS选择器（都写在<\*\*style\*\*>**</\**style\*>****）***：\*\*  

                                

![image](images/cdfe559c-4af4-4877-a408-2a07c200de92.jpg)



**1.标签选择器**  

![image](images/7d89b4ec-ea16-4c04-9f87-b00c732906de.jpg)

**2.类选择器（一个类选择器可以对应多个标签）**  

    **★\*\*\*\*情景一（多个标签使用同一个名称的类选择器，且每个标签使用效果相同）：**  

![image](images/5025b140-1c8a-4e7b-833f-bfc49a10c67f.jpg)

    **★\*\*\*\*情景二（多个标签使用同一个名称的类选择器，且每个类实现效果不同）：**  

![image](images/9973ab27-e395-49f1-b561-ec4e50e5254e.jpg)

    **★\*\*\*\*情景三（一个标签使用多个类选择器）：**  

![image](images/a2689872-78f0-40ce-8f15-4d7f97f14cae.jpg)

**3.ID选择器（ID是唯一的，一个ID选择器只能对应一个标签）**  

![image](images/0de21450-3670-4d83-86e2-6f739415871e.jpg)

**4.全局选择器**  



![image](images/e089136e-cef1-46a6-8764-4b45aa3407b4.jpg)

**5.群组选择器**  

![image](images/b15262e2-d59d-48e1-9ed7-3b29522fdcc2.jpg)

**6.后代选择器**  

    **★\*\*\*\*情景一：**  

![image](images/40793fc1-52c5-4e32-b430-54fc6f5d3560.jpg)

    **★\*\*\*\*情景二：**  

![image](images/e40b5bf0-43db-40cb-aa22-a54aae21a202.jpg)



**例1 标签选择器:**  

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Css样式</title>
    <style type="text/css">
        h1{
            color: red;
        }
        p{
            color: blue;
        }
    </style>
</head>
<body>
    <h1>CSS层叠样式</h1>
    <p>用于定义HTML内容在浏览器中的显示样式</p>
</body>
</html>
```
**例2.1 类选择器**（多个标签使用同一个**名称的**类选择器，且每个标签使用效果相同）\*\*:\*\*  

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Css样式</title>
    <style type="text/css">
        .one{
            color: red;
            font-size: 30px;
        }
    </style>
</head>
<body>
    <h1 class="one">CSS层叠样式</h1>
    <p class="one">用于定义HTML内容在浏览器中的显示样式</p>
</body>
</html>
```
**例2.2 类选择器**（多个标签使用同一个名称的类选择器，且每个类实现效果不同）\*\*:\*\*  

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Css样式</title>
    <style type="text/css">
        p.one{
            color: red;
            font-size: 30px;
        }
        h1.one{
            color: blue;
            font-size: 10px;
        }
    </style>
</head>
<body>
    <h1 class="one">CSS层叠样式</h1>
    <p class="one">用于定义HTML内容在浏览器中的显示样式</p>
</body>
</html>
```
**例2.3 类选择器**（一个标签使用多个类选择器）\*\*:\*\*  

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Css样式</title>
    <style type="text/css">
        .one{
            color: red;

        }
        .three{
            font-size: 10px;
        }

    </style>
</head>
<body>
    <h1 class="one">CSS层叠样式</h1>
    <p class="one thre">用于定义HTML内容在浏览器中的显示样式</p>
</body>
</html>
```
**例3 ID选择器:**  

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Css样式</title>
    <style type="text/css">
        #one{
            color: yellow;
        }

    </style>
</head>
<body>
    <h1 id="one">CSS层叠样式</h1>
    <p>用于定义HTML内容在浏览器中的显示样式</p>
</body>
</html>
```
**伪类：**  



![image](images/d151067c-781a-43a5-be5a-9019d7b30628.jpg)

  

**1.链接伪类**

![image](images/98421125-c326-48c4-b2d3-171aee9b7222.jpg)

![image](images/4bca2890-cda3-4cb1-9576-54b1609e3859.jpg)





**1.伪类的hover和active其他用途**  

![image](images/a04e9003-8aad-4458-a8f0-9480b788d29c.jpg)

**3.伪类**hover和active的**兼容**  

![image](images/5a3c329f-ce42-4391-8c35-d8cd67bf241b.jpg)





**例1 链接伪类:**  

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Css样式</title>
    <style type="text/css">
        a:link{color:blue;} /*未访问状态*/
        a:visited{color: gray;}/*已访问状态*/
        a:hover{color:green;}/*鼠标悬浮状态*/
        a:active{color:orange;}/*激活状态*/

    </style>
</head>
<body>
    <a href="http://www.baidu.com" target="_blank">跳转</h1>
</body>
</html>
```
**例2 伪类hover和active:**  

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Css样式</title>
    <style type="text/css">
        p:hover{color:red;}/*鼠标悬浮状态*/
        p:active{font-size:20px;}/*激活状态*/

    </style>
</head>
<body>
    <p>你好</p>
</body>
</html>
```