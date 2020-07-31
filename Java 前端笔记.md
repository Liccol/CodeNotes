# JAVA 前端笔记
## html基础
html一般都要包括:
<html>
	<body>
		
	</body>

</html>
HTML是由一套标记标签 （markup tag）组成，通常就叫标签

**标签由开始标签和结束标签组成**
<p> 这是一个开始标签
</p> 这是一个结束标签
<p> Hello World </p> 标签之间的文本叫做内容

属性用来修饰标签的
比如要设置一个标题居中：
<h1 align="center">居中标题</h1>
写在开始标签里的 align="center" 就叫属性
align 是属性名
center 是属性值
属性值应该使用双引号括起来
### 超链接
<a>标签即用来显示超链
完整元素是
<a href="跳转到的页面地址">超链显示文本</a>
### 图像
<img src="https://how2j.cn/example.gif"/>
属性值里可以是本地文件地址
### 块（批量布局）
<div>
<span>
这两种标签都是布局用的，这种标签本身没有任何显示效果，通常是用来结合css进行页面布局。

 <img style="margin-left:50px" src="https://how2j.cn/example.gif"/>
  <br/>
 <img style="margin-left:50px" src="https://how2j.cn/example.gif"/>


<div style="margin-left:100px" >
 <img src="https://how2j.cn/example.gif"/>
  <br/>
 <img src="https://how2j.cn/example.gif"/>
</div>

加入div，对于内容的批量布局设置会方便很多

div是块元素，即自动换行
常见的块元素还有h1,table,p
span是内联元素，即不会换行
常见的内联元素还有img,a,b,strong
### 字体
<font color="blue" size="+2">蓝色大2号字体</font>
### 内链框架
通过内联框架 可以实现在网页中“插入”网页
<iframe src="https://how2j.cn/" width="600px" height="400px">
</iframe>

## html表单元素

### 文本框
<input type="text" placeholder="请输入账号" size="15">

多行文本

<textarea cols="30" rows="8">123456789012345678901234567890
1234567890
1234567890
1234567890
1234567890
1234567890
1234567890
1234567890</textarea>
### 带星号的密码框
<input type="password" placeholder="enter your password" size="20">
### 选择，默认选择
<!-- name表示以下的几个内容都是同一个分组，分组内只能单选，value表示这个分组里的值，因为是单选，所以最后只有一个value给后段数据-->
学习java<input type="radio" name="activity" value="学习java" > <br/>
<!-- radio表示选择图标，checked属性表示默认选择 -->
东京热<input type="radio" checked="checked"  name="activity" value="tokyohot" > <br/>
dota<input type="radio" name="activity" value="dota" > <br/>
LOL<input type="radio" name="activity"  value="lol"> <br/>
## 多选框
<!-- checkbox来设置成复选框，可以多选-->
学习java<input type="checkbox" value="学习java" > <br/>
东京热<input type="checkbox" checked="checked"  name="activity" value="tokyohot" > <br/>
dota<input type="checkbox" value="dota" > <br/>
LOL<input type="checkbox" value="lol"> <br/>
### 下拉列表进行选择
<!-- 	multiple选中表示可以多选-->
<select size="3" multiple="multiple">
 <option >苍老师</option>
 <option >高树玛利亚</option>
 <option >遥美</option>
</select>
### 提交按钮
<form action="/study/login.jsp" method="get">
账号：<input type="text" name="name"> <br/>
密码：<input type="password" name="password" > <br/>
<!--使用type=submit表示是一个提交按钮-->
<input type="submit" value="登陆">
<!--使用type=reset表示是一个提交按钮-->
<input type="reset" value="重置">
</form>

## css
css的使用是为了更方便的进行多重网页的格式和布局控制，实际上是引入了style标签，对全文内容进行格式控制，配合选择器。
### 语法
css的语法
**selector {property: value} **
即 选择器{属性:值}
学习css即学习有哪些选择器，哪些属性以及可以使用什么样的值

示例：
// 选择所有的P，然后改成红色
<style>
p{
   color:red;
}
</style>
<p>这是一个P</p>
<p>这是一个P</p>
<p>这是一个P</p>
<p>这是一个P</p>
// 或者可以单独修改，采用style
<p style="color:red">这是style为红色的</p>
<p>这是一个没有style的p</p>
### 三类选择器
选择器主要分3种：
1. 元素选择器：元素选择器通过标签名选择元素在实例中，所有的p都被设置成红色
<style>
p{
  color:red;
}
</style>
<p>p元素</p>
<p>p元素</p>

2. id选择器：通过id选择元素，一个元素的id应该是唯一的。另一个元素不应该重复使用。而id则是在标签头位置设置。
// **注意id选择时使用的是  # + id名**
<style>
p{
  color:red;
}
#p1{
  color:blue;
}
#p2{
  color:green;
}
</style>

<p>没有id的p</p>
<p id="p1">id=p1的p</p>
<p id="p2">id=p2的p</p>
3. 类选择器：当需要多个元素，都使用同样的css的时候，就会使用类选择器
// **注意class时使用的是  . + class名**
<style>
.pre{
  color:blue;
}
.after{
  color:green;
}
</style>
 
<p class="pre">前3个</p>
<p class="pre">前3个</p>
<p class="pre">前3个</p>
 
<p class="after">后3个</p>
<p class="after">后3个</p>
<p class="after">后3个</p>
4. 更准确的选择，同时根据元素名和class来选择
<style>
// 注意是p.blue，表示的是对标签为p且class为blue的内容进行修改。
p.blue{
  color:blue;
}
</style>
 
<p class="blue">class=blue的p</p>
 
<span class="blue">class=blue的span</span>
### 尺寸设置
<style>
p#percentage{
  width:50%;
  height:50%;
  background-color:pink;
}
p#pix{
 /* 按比例和按尺寸的不同实际上就是一个用百分比，一个用实际的px*/
  width:180px;
  height:50px;
  background-color:green;
}
</style>

<p id="percentage"> 按比例设置尺寸50%宽 50%高</p>
<p id="pix"> 按象素设置尺寸  180px宽 50 px高</p>
### 文本设置
1. 对齐等格式
color	文字颜色	
text-align	对齐	
text-decoration	文本修饰	
line-height	行间距	
letter-spacing	字符间距	
word-spacing	单词间距	
text-indent	首行缩进	
text-transform	大小写	
white-space	空白格

<style>
div#left{
  text-align:left;
}
div#right{
  text-align:right;
}
div#center{
  text-align:center;
}
span#right{
  text-align:right;
}
</style>

<div id="left">
左对齐
</div>
<div id="right">
右对齐
</div>
<div id="center">
居中
</div>
<span id="right">
span看不出右对齐效果
2. 字体大小
font-size	尺寸	
font-style	风格	
font-weight	粗细	
font-family	字库	
font	声明在一起

<style>
p.big{
  font-size:30px;
  font-style:normal;
  font-weight:bold;
}
p.small{
  font-size:50%;
}
p.small2{
  font-size:0.5em;
}  
</style>

<p >正常大小</p>
<p class="big">30px大小的文字</p>
<p class="small">50%比例的文字</p>
<p class="small2">0.5em 等同于 50%比例的文字</p>
### 边框模型
// 对边框进行统一设置
<style>
.box{
    width:70px;
    padding:5px;
    margin: 10px;
}

div{
   border:1px solid gray;
   font-size:70%;
}
</style>

<div>
 其他元素
</div>

<div class="box">
   内容宽度70px <br><br>
   内边距：5px <br> <br>
   外边距：10px <br>
</div>

<div>
 其他元素
</div>
### 隐藏
<style>
div.d{
// 隐藏一个元素，这个元素将不再占有原空间 “坑” 让出来了
  display:none;
}

div.v{
// 隐藏一个元素，这个元素继续占有原空间，只是“看不见”
  visibility:hidden;
}
</style>

<div>可见的div1</div>
<div class="d">隐藏的div2,使用display:none隐藏</div>
<div>可见的div3</div>
<div class="v">隐藏的div4,使用visibility:hidden隐藏</div>
<div>可见的div5</div>
### css整合成文件
css的内容都是放在style这个标签里面的，如果写在html里面，会看起来很麻烦，所以可以整合到一个文件里去
具体操作如下：
1. 创建style.css，内容：（其中就不需要写style标签了，文件名默认包含进去这个style标签）
.p1{
  color:red;
}
.span1{
  color:blue;
}
2.在html中包含该文件，使用link，type:text/css来指明链接一个css文件,可以是相对位置，也可以是绝对位置
<link rel="stylesheet" type="text/css" href="./style.css" />
### css优先级
1. css文件和html内部的css内容可能重复，这就涉及到优先级的问题
外部文件style.css的内容和html中style标签的内容重复，那么优先使用html中最后出现的一个
2. style标签上与其他标签自己的style属性冲突
优先使用style属性
3. 如果样式上增加了!important，则优先级最高，甚至高于style属性
<style>
.p1{
  color:green !important;
}
</style>

<p class="p1" style="color:red">p1 颜色是绿色，优先使用!important样式</p>
### 布局中的定位
1. 绝对定位
设置了绝对定位的元素，相当于该元素被从原文档中删除了
所以”正常文字4“会紧接着出现在 ”正常文字2“后面，而不会留下空档
<style>
p.abs{
  position: absolute;
  left: 150px;
  top: 50px;
}

</style>
<p >正常文字2</p>
<p class="abs" >绝对定位的文字3</p>
<p >正常文字4</p>

1.1 绝对定位基于最近的定位的父容器
也就是，例如某个绝对定位的对象，他存在于某个父容器如块div中，他会基于div的位置进行绝对定位。
<style>
p.abs{
  position: absolute;
  left: 100px;
  top: 50px;
}
.absdiv{
  position: absolute;
  left: 150px;
  top: 50px; 
  width:215px;
  border: 1px solid blue;
}
</style>

<div>
<p >正常文字a</p>
<p >正常文字b</p>
</div>

<div class="absdiv">
这是一个定位了的div
<p class="abs" >绝对定位的文字</p>
</div>

-->效果是以 这是个定位了的div为原点，然后进行原点定位
--> 如果没有父容器，那么他就会以整个body原点为基准点
-->若重叠了，可以用z-index来标定图层上下级