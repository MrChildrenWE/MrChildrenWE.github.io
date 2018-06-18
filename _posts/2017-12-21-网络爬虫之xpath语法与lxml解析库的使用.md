---
layout:     post   				    # 使用的布局（不需要改）
title:      网络爬虫之lxml解析库的使用				# 标题 
subtitle:   xpath 语法与lxml 的使用 #副标题
date:       2017-12-21			# 时间
author:     Mr.Children						# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 网络爬虫
---

#### Xpath 的语法  

###### 1.实例文档  

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<bookstore>
<book>
  <title lang="eng">Harry Potter</title>
  <price>29.99</price>
</book>
<book>
  <title lang="eng">Learning XML</title>
  <price>39.95</price>
</book>
</bookstore>
```

###### 2.选取节点 

| 表达式 | 描述 | 
| ------- | --------| 
| nodename | 选取此节点的所有子节点 |  
| /       |  从根节点选取    |
| //    |  从匹配选择的当前节点选择文档中的节点，而不考虑他们的位置 |
| .   | 选取当前节点  |
| .. | 选取当前节点的父节点 |
| @ | 选取属性  |   

###### 3.实例 

|路径表达式 | 结果|  
| --------- | ---  |   
| bookstore | 选取bookstore元素的所有子节点 | 
|/bookstore | 选取根元素bookstore |  
|bookstore/book |选取bookstore 的子元素的所有book 元素 |  
| //book| 选取所有book子元素，而不管它们在文档中的位置 |  
| bookstore//book | 选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置 |  
| //@lang | 选取名为lang 的所有属性 |

###### 4.谓语实例（用来查找某个特定的节点或者包含某个指定的值的节点）

|  路径表达式 | 结果  |   
| --------- | ------ |
| /bookstore/book[1] |  选取属于 bookstore 子元素的第一个 book 元素 |
| /bookstore/book[last()]  | 选取属于 bookstore 子元素的最后一个 book 元素 |   
| /bookstore/book[last()-1]  | 选取属于 bookstore 子元素的倒数第二个 book 元素 |   
|  /bookstore/book[position()<3] | 选取最前面的两个属于 bookstore 元素的子元素的 book 元素 |   
| //title[@lang]  | 选取所有拥有名为 lang 的属性的 title 元素 |  
| //title[@lang='eng']  |选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性  |  
| /bookstore/book[price>35.00]  | 选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00 |  
| /bookstore/book[price>35.00]/title  | 选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00 |   

###### 5.选取未知节点  

|  通配符 | 描述 |   
| ------ | ---- |  
| /bookstore/* | 选取bookstore元素的所有子元素  |
| //*          | 选取文档中的所有元素     |  
| //title[@*]  | 选取所有带有属性的title 元素 |  

###### 6.路径的表达式  

```
#绝对路径  
/step/step/
#相对路径  
step/step/
```  
#### lxml的用法  

``` 
#lxml 会自动修正html 的代码 
from lxml import etree
text = '''
<div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html">third item</a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a>
     </ul>
 </div>
'''
html = etree.HTML(text)
result = etree.tostring(html)
print(result)  
```  

###### 1.读取文件  

```
#新建文件  hello.html  
<div>
    <ul>
         <li class="item-0"><a href="link1.html">first item</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-inactive"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-1"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
     </ul>
 </div>
from lxml import etree   
html = etree.parse("hello.html")  
result = etree.tostring(html,pretty_print=True)  
print(result)
```  

###### 2.获取所有li 标签  

```
from lxml import etree  
html = etree.parse('hello.html')  
print type(html)  
result = html.xpath('//li')
print (result)  # 返回的是列表  
print len(result)  
print type(result)  
print type(result[0])   
#获取li标签的所有class  
result = html.xpath('//li/@class')  
print (result)  
 ['item-0', 'item-1', 'item-inactive', 'item-1', 'item-0']  
#获取li 标签下href 属性为link1.html的<a>标签  
result = html.xpath("//li/a[@href="link1.html"]")
print (result[0].text)  
#获取li 标签下的所有<span> 标签 双斜杠,不是子元素 
result = html.xpath('//li//span')
result = html.xapth('//li/a//@class')  
#获取最后一个li 标签的<a> 的href  
result = html.xpath('//li[last()]/a/@href')  
#获取倒数第二个元素的内容  
result = html.xpath('//li[last()-1]/a')  
获取class 为bold 的标签名  
result = html.xpath('//*[@class='bold']')  
print (result[0].tag)  
```