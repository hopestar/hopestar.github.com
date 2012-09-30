---
layout: default
title: html 学习笔记
---
虽然很多语言都是可以转换为html的标准格式，但是很多操作都是需要对html格式进行直接操作，所以基本格式还是要了解的。
###学习资料：w3c
还是拿个栗子来讲解容易点。    
举个栗子：    
<h1>基本的 HTML 标签</h1>
<div  id="tpn">
<ul class="prenext">
<li class="pre"><a href="" title=""></a></li>
<li class="next"><a href="" title=""></a></li>
</ul>
</div>

<div id="intro">
<p><strong>本章向您讲解 HTML 标签的概念，通过实例向您演示最常用的 HTML 标签。</strong></p>
<p class="tip"><span>提示：</span>不要担心本章中您还没有学过的例子，您将在下面的章节中学到它们。</p>
<p class="tip"><span>提示：</span>学习 HTML 最好的方式就是边学边做实验。我们为您准备了很好的 HTML 编辑器。使用这个编辑器，您可以任意编辑一段 HTML 源码，然后单击 TIY 按钮来查看结果。</p>
</div>

<div>
<h2>什么是 HTML 标签</h2>

<ul>
<li>HTML 文档和 HTML 元素是通过 HTML 标签进行标记的</li>
<li>HTML 标签由开始标签和结束标签组成</li>
<li>开始标签是被括号包围的元素名</li>
<li>结束标签是被括号包围的斜杠和元素名</li>
<li>某些 HTML 元素没有结束标签，比如 &lt;br /&gt;</li>
</ul>

<p class="note"><span>注释：</span>开始标签的英文翻译是 start tag 或 opening tag，结束标签的英文翻译是 end tag 或 closing tag。</p>
</div>


<div>
<h2>HTML 标题</h2>

<p>HTML 标题（Heading）是通过 &lt;h1&gt; - &lt;h6&gt; 等标签进行定义的。</p>

<h3>实例</h3>

<pre>
&lt;h1&gt;This is a heading&lt;/h1&gt;
&lt;h2&gt;This is a heading&lt;/h2&gt;
&lt;h3&gt;This is a heading&lt;/h3&gt;
</pre>

<p><a href="/tiy/t.asp?f=html_headers">亲自试一试</a></p>
</div>


<div>
<h2>HTML 段落</h2>

<p>HTML 段落是通过 &lt;p&gt; 标签进行定义的。</p>

<h3>实例</h3>

<pre>
&lt;p&gt;This is a paragraph.&lt;/p&gt;
&lt;p&gt;This is another paragraph.&lt;/p&gt;
</pre>

<p><a href="/tiy/t.asp?f=html_paragraphs1">亲自试一试</a></p>
</div>


<div>
<h2>HTML 链接</h2>
<p>HTML 链接是通过 &lt;a&gt; 标签进行定义的。</p>
<h3>实例</h3>
<pre>&lt;a href="http://www.w3school.com.cn"&gt;This is a link&lt;/a&gt;</pre>
<p><a href="/tiy/t.asp?f=html_basic_link">亲自试一试</a></p>
<p class="note"><span>注释：</span>在 href 属性中指定链接的地址。</p>
<p>（您将在本教程稍后的章节中学习更多有关属性的知识）。</p>
</div>

<div>
<h2>HTML 图像</h2>
<p>HTML 图像是通过 &lt;img&gt; 标签进行定义的。</p>
<h3>实例</h3>
<pre>&lt;img src="w3school.jpg" width="104" height="142" /&gt;</pre>
<p><a href="/tiy/t.asp?f=html_basic_img">亲自试一试</a></p>
<p class="note"><span>注释：</span>图像的名称和尺寸是以属性的形式提供的。</p>
</div>
