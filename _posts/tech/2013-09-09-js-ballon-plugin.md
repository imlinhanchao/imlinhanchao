---
author: hancel.lin
date: 2013-09-09
title: 一个气泡提示的Javascript控件
tags: javascript,Web,控件
images: /blog/img/js-ballon-plugin/01.jpg
category: tech
layout: default
---
源起
---
![Chrome气泡提示][1]

某日，忽的想写个js小控件。功能很简单，就是可以在文本框下面显示一个气泡提示，如上图。图是Chrome里截来的，是Chrome原生的提示样式。只要在文本框启用『required』，提交时内容为空时就会出现如图提示。

![气泡提示HTML原型][2]

开始
---

首先第一步，该是要构建一个提示文字的HTML模型，那么弹出提示时就可以被重复构建了（如上图）。
{% highlight html %}
<div class="megbox">
    <div class="megbox_top"></div>
    <div class="megbox_meg">
        <div class="megbox_txt">提示文字..</div>
    </div>
</div>
{% endhighlight %}

HTML模型包含两个部分，提示文字和一个啥也没有是div，那个div就是用来显示提示消息上方的小三角的~因此，我们还需要一些CSS来定义赋予样式。

{% highlight css %}
.megbox{
    position: absolute;
    background-color: #FFF;
    border: 1px solid #a4acb5;
    padding: 5px 10px;
    border-radius: 5px;
    z-index: 100;
}
.megbox_top{
    width: 13px;
    height: 7px;
    position: absolute;
    background: url(./images/up.gif) no-repeat;
    top: -7px;
}
.megbox_meg{
    padding-left: 25px;
    background: url(./images/warning.gif) no-repeat;
    line-height: 20px;
}
.megbox_txt{}/* 暂且保留，说不定以后加点什么样式~~ */
{% endhighlight %}

CSS一共需要调用到两张背景图：![尖端][3]　![warning][4]

因为气泡提示基本上是悬浮在文本框上方的，所以不会被计入正常流中，因此通过「position: absolute」属性使其脱离正常流。

接下来就是Javascript的部分了。因为本人大学学的是C++，所以对面向对象比较钟情，所以就用面向对的方法来开发这个js控件。

首先，定义一下我们需要那些属性和方法：

属性：

element：气泡指向的文本框元素，姑且称之为目标元素吧~；

message：消息提示的内容；

id：说不定我们需要弹出好多个提示框，所以我们需要一个唯一标识来区隔它们~；

方法：

Show：显示气泡提示；

Remove：移除气泡；

OK，首先写个构造函数，这个类就叫MessageBox吧~

{% highlight javascript %}
MessageBox = function(element, id, message)
{
    // Init value
    this.message = "";
    this.element = undefined;   
    this.id = "";

    this.message = message;
    this.element = element;
    this.id = id;
};
{% endhighlight %}
接着就是完成两个方法了~

Show——首先要解决两个问题：

插入DOM元素；

显示在哪儿？

第一个问题好解决，查一查[W3School](http://www.w3school.com.cn/xmldom/dom_nodes_create.asp)就O了~

第二个，因为我们是用绝对定位，而且是要显示在目标元素附近。因此就需要知道目标元素的位置。这个嘛…就要Google一下了。

我参照了这位大牛的函数——[阮一峰](http://www.ruanyifeng.com/blog/2009/09/find_element_s_position_using_javascript.html)。

略做修改后，如下：

{% highlight javascript %}
document.getElementView = function (element)
{
    if(element != document)
        return {
            width: element.offsetWidth,
            height: element.offsetHeight
        }
    if (document.compatMode == "BackCompat"){
        return {
            width: document.body.clientWidth,
            height: document.body.clientHeight
        }
    } else {
        return {
            width: document.documentElement.clientWidth,
            height: document.documentElement.clientHeight
        }
    }
};

document.getElementLeft = function (element)
{
    var actualLeft = element.offsetLeft;
    var current = element.offsetParent;
    while (current !== null){
        actualLeft += current.offsetLeft;
        current = current.offsetParent;
    }
    return actualLeft;
};

document.getElementTop = function (element)
{
    var actualTop = element.offsetTop;
    var current = element.offsetParent;
    while (current !== null){
        actualTop += current.offsetTop;
        current = current.offsetParent;
    }
    return actualTop;
};
{% endhighlight %}

把他们都作为document方法加进入了~这样似乎不太好，最安全的做法应该是作为MessageBox的私有方法。不过个人喜欢啦~

Remove的话就是把创建的元素删除而已~

最后方法定义如下：

{% highlight javascript %}
MessageBox.prototype = {

    constructor : MessageBox, // 声明构造函数

    Show : function()
    {
        if(!this.element) return false;
        if(this.element.box)
            this.element.box.Remove(true);
        var megbox = document.createElement("div");
        megbox.className = "megbox";
        megbox.id = "megbox_" + this.id;    //把id加上前缀，作为气泡的id
        var megbox_top = document.createElement("div");
        megbox_top.className = "megbox_top";
        var megbox_meg = document.createElement("div");
        megbox_meg.className = "megbox_meg";
        var megbox_txt = document.createElement("div");
        megbox_txt.className = "megbox_txt";
        var megs=document.createTextNode(this.message);

        megbox.appendChild(megbox_top);
        megbox.appendChild(megbox_meg);
        megbox_meg.appendChild(megbox_txt);
        megbox_txt.appendChild(megs);
        this.element.box = this;

        document.getElementsByTagName("body")[0].appendChild(megbox);

        var node_view = document.getElementView(this.element);
        var node_top = document.getElementTop(this.element);
        var node_left = document.getElementLeft(this.element);

        megbox.style.top = (node_top + node_view.height + 5) + "px";
        megbox.style.left = node_left + "px";

        return true;
    },

    Remove : function()
    {
        var id = this.id;
        var node = document.getElementById("megbox_" + id);
        if(node)
        {
            node.parentNode.removeChild(node);
            return true;
        }
        return false;
    }   
};
{% endhighlight %}

因为提示框显示在目标元素下方，因此提示框绝对定位的

top = node_top + node_view.height + 5 ;（如下图）

加上5px是为了避免提示框贴在文本框底部。

![示意][5]

如此，气泡提示控件就完成了，调用时如下：

{% highlight javacript %}
var test = document.getElementById("test");
var Box = new MessageBox(test, 1, "Test Message..");
Box.Show(); // Show the MessageBox

// -----------------------------------

if(Box instanceof MessageBox) Box.Remove(); // Remove MessageBox
{% endhighlight %}
最后
---
最后，附上增强版MessageBox——[下载地址››](http://pan.baidu.com/share/link?shareid=1309283857&uk=1460016148)

  [1]: /img/js-ballon-plugin/pic_01.jpg
  [2]: /img/js-ballon-plugin/pic_02.jpg
  [3]: /img/js-ballon-plugin/pic_03jpg
  [4]: /img/js-ballon-plugin/pic_04.jpg
  [5]: /img/js-ballon-plugin/pic_05.jpg
