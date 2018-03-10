
> JavaScript 与 HTML 之间的交互是通过事件实现的。事件，就是文档或浏览器窗口中发生的一些
特定的交互瞬间。可以使用侦听器（或处理程序）来预订事件，以便事件发生时执行相应的代
码。这种在传统软件工程中被称为观察员模式的模型，支持页面的行为（JavaScript 代码）与页
面的外观（HTML 和 CSS 代码）之间的松散耦合。

## 13.1 事件流
&emsp;&emsp;事件流描述的是从页面中接收事件的顺序。但有意思的是， IE 和 Netscape 开发团队居然提出了差不多是完全相反的事件流的概念。 IE 的事件流是事件冒泡流，而 Netscape Communicator 的事件流是事件捕获流。
### 13.1.1 事件冒泡
&emsp;&emsp;IE 的事件流叫做事件冒泡（event bubbling），即事件开始时由最具体的元素（文档中嵌套层次最深
的那个节点）接收，然后逐级向上传播到较为不具体的节点（文档）。以下面的 HTML 页面为例：

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Event Bubbling Example</title>
    </head>
    <body>
        <div id="myDiv">Click Me</div>
    </body>
</html>
```
&emsp;&emsp;如果你单击了页面中的<div>元素，那么这个 click 事件会按照如下顺序传播：

- (1) \<div>
- (2) <body>
- (3) \<html>
- (4) document

### 13.1.2 事件捕获
&emsp;&emsp;Netscape Communicator 团队提出的另一种事件流叫做事件捕获（event capturing）。事件捕获的思想
是不太具体的节点应该更早接收到事件，而最具体的节点应该最后接收到事件。**事件捕获的用意在于在
事件到达预定目标之前捕获它**。如果仍以前面的 HTML 页面作为演示事件捕获的例子，那么单击<div>
元素就会以下列顺序触发 click 事件。
- (1) document
- (2) \<html>
- (3) <body>
- (4) \<div>

### 13.1.3 DOM 事件流
&emsp;&emsp;“DOM2级事件”规定的事件流包括三个阶段：事件捕获阶段、处于目标阶段和事件冒泡阶段。首
先发生的是事件捕获，为截获事件提供了机会。然后是实际的目标接收到事件。最后一个阶段是冒泡阶
段，可以在这个阶段对事件做出响应。
![image](https://github.com/willingtolove/ReadingNotes/blob/master/professional-js/chapter13-Events/img/13.1.3-1.png?raw=true)

&emsp;&emsp;在 DOM 事件流中，实际的目标（<div>元素）在捕获阶段不会接收到事件。这意味着在捕获阶段，
事件从 document 到<html>再到<body>后就停止了。下一个阶段是“处于目标”阶段，于是事件在<div>
上发生，并在事件处理中被看成冒泡阶段的一部分。然后，冒泡阶段发生，事件又传播回文档。

&emsp;&emsp;多数支持 DOM 事件流的浏览器都实现了一种特定的行为；即使“DOM2 级事件”规范明确要求捕
获阶段不会涉及事件目标，但 IE9、 Safari、 Chrome、 Firefox 和 Opera 9.5 及更高版本都会在捕获阶段触
发事件对象上的事件。结果，就是有两个机会在目标对象上面操作事件。
## 13.2 事件处理程序
&emsp;&emsp;事件就是用户或浏览器自身执行的某种动作。诸如 click、 load 和 mouseover，都是事件的名字。
而响应某个事件的函数就叫做事件处理程序（或事件侦听器）。事件处理程序的名字以"on"开头，因此
click 事件的事件处理程序就是 onclick， load 事件的事件处理程序就是 onload。为事件指定处理
程序的方式有好几种。
### 13.2.1 HTML 事件处理程序
&emsp;&emsp;某个元素支持的每种事件，都可以使用一个与相应事件处理程序同名的 HTML 特性来指定。这个
特性的值应该是能够执行的 JavaScript 代码。

```html
<input type="button" value="Click Me" onclick="alert('Clicked')" />
```
&emsp;&emsp;由于这个值是 JavaScript，因此不能在其中使用未经转义的 HTML 语法字符，
例如和号（&）、双引号（""）、小于号（<）或大于号（>）。为了避免使用 HTML 实体，这里使用了单
引号。如果想要使用双引号，那么就要将代码改写成如下所示：

```html
<input type="button" value="Click Me" onclick="alert(&quot;Clicked&quot;)" />
```
&emsp;&emsp;在 HTML 中定义的事件处理程序可以包含要执行的具体动作，也可以调用在页面其他地方定义的
脚本，如下面的例子所示：

```html
<script type="text/javascript">
    function showMessage(){
        alert("Hello world!");
    }
</script>
<input type="button" value="Click Me" onclick="showMessage()" />
```
&emsp;&emsp;事件处理程序中的代码在执行时，有权访问全局作用
域中的任何代码。
&emsp;&emsp;这样指定事件处理程序具有一些独到之处。首先，这样会创建一个封装着元素属性值的函数。这个
函数中有一个局部变量 event，也就是事件对象。

```html
<!-- 输出 "click" -->
<input type="button" value="Click Me" onclick="alert(event.type)">
```
&emsp;&emsp;通过 event 变量，可以直接访问事件对象，你不用自己定义它，也不用从函数的参数列表中读取。
在这个函数内部， this 值等于事件的目标元素，例如：

```HTML
<!-- 输出 "Click Me" -->
<input type="button" value="Click Me" onclick="alert(this.value)">
```
&emsp;&emsp;关于这个动态创建的函数，另一个有意思的地方是它扩展作用域的方式。在这个函数内部，可以像
访问局部变量一样访问 document 及该元素本身的成员。这个函数使用 with 像下面这样扩展作用域：

```javascript
function(){
    with(document){
        with(this){
            //元素属性值
        }
    }
}
```

```html
<!-- 输出 "btn" -->
<input type="button" id="btn" value="Click Me" onclick="alert(id)"/>
<!-- 输出 "btn" -->
<input type="button" id="btn" value="Click Me" onclick="alert(getElementById('btn').id)"/>
```
&emsp;&emsp;如果当前元素是一个表单输入元素，则作用域中还会包含访问表单元素（父元素）的入口，这个函
数就变成了如下所示：

```javascript
function(){
    with(document){
        with(this.form){
            with(this){
                //元素属性值
            }
        }
    }
}
```
&emsp;&emsp;实际上，这样扩展作用域的方式，无非就是想让事件处理程序无需引用表单元素就能访问其他表单
字段。例如：

```html
<!-- 输出 "xiaoming" -->
<form method="post">
    <input type="text" name="username" value="xiaoming">
    <input type="button" value="Echo Username" onclick="alert(username.value)">
</form>
```
&emsp;&emsp;在 HTML 中指定事件处理程序有两个缺点:

&emsp;&emsp;首先，如果用户在页面解
析事件处理程序(showMessage())之前就单击了按钮，就会引发错误;

&emsp;&emsp;其次，这样扩展事件处理程序的作用域链在不同浏览器中会导致不同结果。不同 JavaScript
引擎遵循的标识符解析规则略有差异，很可能会在访问非限定对象成员时出错。(不懂)

&emsp;&emsp;最后，如果要更换事
件处理程序，就要改动两个地方： HTML 代码和 JavaScript 代码。

### 13.2.2 DOM0 级事件处理程序
&emsp;&emsp;通过 JavaScript 指定事件处理程序的传统方式，就是将一个函数赋值给一个事件处理程序属性。一是简单，二是具有跨浏览器的优势。要使用 JavaScript 指定事件处理程序，首先必须取得一
个要操作的对象的引用。

&emsp;&emsp;每个元素（包括 window 和 document）都有自己的事件处理程序属性，这些属性通常全部小写，
例如 onclick。将这种属性的值设置为一个函数，就可以指定事件处理程序，如下所示：

```JavaScript
var btn = document.getElementById("myBtn");
btn.onclick = function(){
    alert("Clicked");
};
```
&emsp;&emsp;使用 DOM0 级方法指定的事件处理程序被认为是元素的方法。因此，这时候的事件处理程序是在
元素的作用域中运行；换句话说，程序中的 this 引用当前元素。

```JavaScript
var btn = document.getElementById("myBtn");
btn.onclick = function(){
    alert(this.id); //"myBtn"
    alert(id); //ReferenceError
    alert(getElementById('myBtn').id);//ReferenceError
};
```
&emsp;&emsp;以这种方式添加的事件处理程序会在事件流的冒泡阶段被处理。

&emsp;&emsp;删除通过 DOM0 级方法指定的事件处理程序:

```javascript
btn.onclick = null; //删除事件处理程序
```
&emsp;&emsp;用这种方法也可以删除使用 HTML 指定的事件处理程序。
### 13.2.3 DOM2 级事件处理程序
&emsp;&emsp;“DOM2 级事件” 定义了两个方法，用于处理指定和删除事件处理程序的操作： addEventListener()
和 removeEventListener()。所有 DOM 节点中都包含这两个方法，并且它们都接受 3 个参数：要处
理的事件名、作为事件处理程序的函数和一个布尔值。最后这个布尔值参数如果是 true，表示在捕获
阶段调用事件处理程序；如果是 false，表示在冒泡阶段调用事件处理程序。

```JavaScript
var btn = document.getElementById("myBtn");
btn.addEventListener("click", function(){
    alert(this.id);//myBtn
}, false);
```
&emsp;&emsp;与 DOM0 级方法一样，这里添加的事件处理程序也是在其依附的元素的作用域
中运行。使用 DOM2 级方法添加事件处理程序的主要好处是可以添加多个事件处理程序。来看下面的例子。

```html
<button id='myBtn' onclick="alert('click1')">
    Click
</button>
```
```JavaScript
var btn = document.getElementById("myBtn");
btn.addEventListener("click", function(){
    alert('click2');
}, false);
btn.addEventListener("click", function(){
    alert("click3");
}, false);
```
&emsp;&emsp;上面例子的结果为：依次弹出click1、click2、click3；

&emsp;&emsp;通过 addEventListener()添加的事件处理程序只能使用 removeEventListener()来移除；移
除时传入的参数与添加处理程序时使用的参数相同。这也意味着通过 addEventListener()添加的匿名函数将无法移除。
&emsp;&emsp;传入 removeEventListener()中的事件处理程序函数必须与传入
addEventListener()中的相同，如下面的例子所示。
```JavaScript
var btn = document.getElementById("myBtn");
    var handler = function(){
    alert(this.id);
};
btn.addEventListener("click", handler, false);
btn.removeEventListener("click", handler, false); //有效！
```
&emsp;&emsp;大多数情况下，都是将事件处理程序添加到事件流的冒泡阶段，这样可以最大限度地兼容各种浏览
器。最好只在需要在事件到达目标之前截获它的时候将事件处理程序添加到捕获阶段。如果不是特别需
要，我们不建议在事件捕获阶段注册事件处理程序。

    IE9、 Firefox、 Safari、 Chrome 和 Opera 支持 DOM2 级事件处理程序。
### 13.2.4 IE 事件处理程序

&emsp;&emsp;IE 实现了与 DOM 中类似的两个方法： attachEvent()和 detachEvent()。这两个方法接受相同
的两个参数：事件处理程序名称与事件处理程序函数。由于 IE8 及更早版本只支持事件冒泡，所以通过
attachEvent()添加的事件处理程序都会被添加到冒泡阶段。

```js
var btn = document.getElementById("myBtn");
btn.attachEvent("onclick", function(){
    alert("Clicked");
});
```
&emsp;&emsp;注意， attachEvent()的第一个参数是"onclick"，而非 DOM 的 addEventListener()方法中的"click"。

&emsp;&emsp;在 IE 中使用 attachEvent()与使用 DOM0 级方法的主要区别在于事件处理程序的作用域。在使
用 DOM0 级方法的情况下，事件处理程序会在其所属元素的作用域内运行；在使用 attachEvent()方
法的情况下，事件处理程序会在全局作用域中运行，因此 this 等于 window。

```js
var btn = document.getElementById("myBtn");
btn.attachEvent("onclick", function(){
    alert(this === window); //true
});
```
&emsp;&emsp;与 addEventListener()类似， attachEvent()方法也可以用来为一个元素添加多个事件处理程序。
···················
# 这块有点不清楚，暂留白
··················
### 13.2.5 跨浏览器的事件处理程序
&emsp;&emsp;要创建的方法是 addHandler()，它的职责是视情况分别使用 DOM0 级方法、 DOM2 级方法或 IE 方法来添加事件。

```js
var EventUtil = {
    addHandler: function(element, type, handler){
        if (element.addEventListener){
            element.addEventListener(type, handler, false);
        } 
        else if (element.attachEvent){
            element.attachEvent("on" + type, handler);
        }
        else {
            element["on" + type] = handler;
        }
    },
    removeHandler: function(element, type, handler){
        if (element.removeEventListener){
            element.removeEventListener(type, handler, false);
        } 
        else if (element.detachEvent){
            element.detachEvent("on" + type, handler);
        } else {
            element["on" + type] = null;
        }
    }
};
```

```js
var btn = document.getElementById("myBtn");
var handler = function(){
    alert("Clicked");
};
EventUtil.addHandler(btn, "click", handler);
EventUtil.removeHandler(btn, "click", handler);
```
## 13.3 事件对象
&emsp;&emsp;在触发 DOM 上的某个事件时，会产生一个事件对象event，这个对象中包含着所有与事件有关的
信息。包括导致事件的元素、事件的类型以及其他与特定事件相关的信息。例如，鼠标操作导致的事件对象中，会包含鼠标位置的信息，而键盘操作导致的事件对象中，会包含与按下的键有关的信息。所有浏览器都支持 event 对象，但支持方式不同。
### 13.3.1 DOM 中的事件对象
&emsp;&emsp;兼容 DOM 的浏览器会将一个event对象传入到事件处理程序中。无论指定事件处理程序时使用什么方法（DOM0 级或 DOM2 级），都会传入 event 对象。

```js
var btn = document.getElementById("myBtn");
btn.onclick = function(event){
    alert(event.type); //"click"
};
btn.addEventListener("click", function(event){
    alert(event.type); //"click"
}, false);
```

```html
<input type="button" value="Click Me" onclick="alert(event.type)"/>
```
&emsp;&emsp;event 对象包含与创建它的特定事件有关的属性和方法。触发的事件类型不一样，可用的属性和方法也不一样。不过，所有事件都会有下表列出的成员。


属性/方法 | 类 型 | 读/写 | 说 明
---|---|---|---
bubbles | Boolean | 只读 | 表明事件是否冒泡
cancelable | Boolean | 只读 |  表明是否可以取消事件的默认行为
currentTarget |  Element |  只读 |  其事件处理程序当前正在处理事件的那个元素
defaultPrevented |  Boolean |  只读 |  为 true 表 示 已 经 调 用 了 preventDefault()（DOM3级事件中新增）
detail |  Integer |  只读 |  与事件相关的细节信息
eventPhase |  Integer |  只读 |  调用事件处理程序的阶段： 1表示捕获阶段， 2表示“处于目标”， 3表示冒泡阶段
preventDefault() |  Function |  只读 |   取消事件的默认行为。如果cancelable是true，则可以使用这个方法
stopImmediatePropagation() |   Function |   只读 |   取消事件的进一步捕获或冒泡，同时阻止任何事件处理程序被调用（DOM3级事件中新增）
stopPropagation()  |  Function |   只读 |   取消事件的进一步捕获或冒泡。如果bubbles为true，则可以使用这个方法
target  |  Element |   只读 |   事件的目标
trusted  |  Boolean |   只读 |   为true表示事件是浏览器生成的。为false表示事件是由开发人员通过JavaScript创建的（DOM3级事件中新增）
type |   String  |  只读 |   被触发的事件的类型
view |   AbstractView  |  只读 |   与事件关联的抽象视图。等同于发生事件的window对象

#### this,currentTarget,target
&emsp;&emsp;在事件处理程序内部，对象 this 始终等于 currentTarget 的值，而 target 则只包含事件的实
际目标。如果直接将事件处理程序指定给了目标元素，则 this、 currentTarget 和target包含相同
的值。

```js
var btn = document.getElementById("myBtn");
btn.onclick = function(event){
    alert(event.currentTarget === this); //true
    alert(event.target === this); //true
};
```
&emsp;&emsp;这个例子检测了currentTarget和target与this的值。由于click事件的目标是按钮，
因此这三个值是相等的。如果事件处理程序存在于按钮的父节点中（例如div），
那么这些值是不相同的。
```html
<div id='myDiv'>
	Click!
	<input type="button" value="Click Me" id='myBtn'/>
</div>
```

```js
var myDiv = document.getElementById("myDiv");
var myBtn = document.getElementById("myBtn");
myDiv.onclick = function(event){
    alert(event.currentTarget === myDiv); //true
    alert(this === myDiv); //true
    alert(event.target === myBtn); //true
};

```
#### preventDefault
&emsp;&emsp;要阻止特定事件的默认行为，可以使用 preventDefault()方法。例如，链接的默认行为就是在
被单击时会导航到其 href 特性指定的 URL。如果你想阻止链接导航这一默认行为，那么通过链接的
onclick 事件处理程序可以取消它。

```js
var link = document.getElementById("myLink");
link.onclick = function(event){
    event.preventDefault();
};
```
&emsp;&emsp; cancelable 属性设置为 true 的事件，才可以使用 preventDefault()来取消其默认行为。
##### demo-1：preventDefault
&emsp;&emsp;说明：当输入的不是小写字母的时候，阻止input元素的默认行为（改变value值）

```html
<p>请输入一些字母,只允许小写字母.</p>
<input type="text" id="my-textbox"/>
```

```js
function checkName(evt) {
var charCode = evt.charCode;
  if (charCode != 0) {
    if (charCode < 97 || charCode > 122) {
      evt.preventDefault();
      alert("只能输入小写字母." + "\n"+ "charCode: " + charCode + "\n"
      );
    }
  }
}
document.getElementById('my-textbox').addEventListener(
    'keypress', checkName, false
);
```
#### stopPropagation
&emsp;&emsp;stopPropagation()方法用于立即停止事件在 DOM 层次中的传播，即取消进一步的事件捕获或冒泡。例如，直接添加到一个按钮的事件处理程序可以调用 stopPropagation()，从而避免触发注册在 div上面的事件处理程序。

```js
var myDiv = document.getElementById("myDiv");
var btn = document.getElementById("myBtn");
btn.onclick = function(event){
    alert("Clicked");
    event.stopPropagation();
};
myDiv.onclick = function(event){
    alert("Body clicked");
};
```
#### eventPhase
&emsp;&emsp;事件对象的 eventPhase 属性，可以用来确定事件当前正位于事件流的哪个阶段。如果是在捕获阶段调用的事件处理程序，那么 eventPhase 等于 1；如果事件处理程序处于目标对象上，则 eventPhase 等于 2；如果是在冒泡阶段调用的事件处理程序， eventPhase 等于 3。这里要注意的是，尽管“处于目标”发生在冒泡阶段，但 eventPhase 仍然一直等于 2。

```js
var btn = document.getElementById("myBtn");
btn.onclick = function(event){
    alert(event.eventPhase); //2
};
document.body.addEventListener("click", function(event){
    alert(event.eventPhase); //1
}, true);
document.body.onclick = function(event){
    alert(event.eventPhase); //3
};

```

##### demo-1：eventPhase

```html
<div id='myDiv' onclick='alert(event.eventPhase+"on")'>
	Click!
	<input type="button" value="Click Me" id='myBtn'/>
</div>
```

```js
//alert顺序：
/*
    "div捕获阶段-1"，
    "btn处于目标阶段-3"，
    "div冒泡阶段1-3"，
    "div冒泡阶段2-3" 
*/

var div = document.getElementById("myDiv");
var btn = document.getElementById("myBtn");

btn.onclick = function(event){
    alert("btn处于目标阶段-"+event.eventPhase); //btn处于目标阶段-3
};
div.addEventListener("click", function(event){
    alert("div冒泡阶段1-"+event.eventPhase); //"div冒泡阶段1-3"
}, false);
div.addEventListener("click", function(event){
    alert("div捕获阶段-"+event.eventPhase); //"div捕获阶段-1"
}, true);
div.onclick = function(event){
    alert("div冒泡阶段2-"+event.eventPhase); //"div冒泡阶段2-3"
};

```
> 只有在事件处理程序执行期间， event 对象才会存在；一旦事件处理程序执行完成， event 对象就会被销毁。
### 13.3.2 IE 中的事件对象

&emsp;&emsp;在使用 DOM0 级方法添加事件处理程序时， event 对象作为 window 对象的一个属性存在。

```js
var btn = document.getElementById("myBtn");
btn.onclick = function(){
    var event = window.event;
    alert(event.type); //"click"
};
```
&emsp;&emsp;如果事件处理程序是使用 attachEvent()添加的，那么就会有一个 event 对象作为参数被传入事件处理程序函数中。

```js
var btn = document.getElementById("myBtn");
btn.attachEvent("onclick", function(event){
    alert(event.type); //"click"
    alert(window.event.type);//"click"
});
```
&emsp;&emsp;在像这样使用 attachEvent()的情况下，也可以通过 window 对象来访问 event 对象，就像使用DOM0 级方法时一样。不过为方便起见，同一个对象也会作为参数传递。
&emsp;&emsp;如果是通过 HTML 特性指定的事件处理程序，那么还可以通过一个名叫 event 的变量来访问 event对象（与 DOM 中的事件模型相同）。

```html
<input type="button" value="Click Me" onclick="alert(event.type)">
```
&emsp;&emsp;IE 的 event 对象同样也包含与创建它的事件相关的属性和方法。其中很多属性和方法都有对应的或者相关的 DOM 属性和方法。与 DOM 的 event 对象一样，这些属性和方法也会因为事件类型的不同而不同，但所有事件对象都会包含下表所列的属性和方法。

属性/方法| 类 型| 读/写| 说 明
---|---|---|---
cancelBubble| Boolean| 读/写| 默认值为false，但将其设置为true就可以取消事件冒泡（与DOM中的stopPropagation()方法的作用相同）
returnValue| Boolean| 读/写| 默认值为true，但将其设置为false就可以取消事件的默认行为（与DOM中的preventDefault()方法的作用相同）
srcElement| Element| 只读| 事件的目标（与DOM中的target属性相同）
type| String| 只读| 被触发的事件的类型

&emsp;&emsp;因为事件处理程序的作用域是根据指定它的方式来确定的，所以不能认为 this 会始终等于事件目标。故而，最好还是使用 event.srcElement 比较保险。

```js
var btn = document.getElementById("myBtn");
btn.onclick = function(){
    alert(window.event.srcElement === this); //true
};
btn.attachEvent("onclick", function(event){
    alert(event.srcElement === this); //false, 这里的this指向window
});
```
&emsp;&emsp;returnValue 属性相当于 DOM 中的 preventDefault()方法，它们的作用都是取消给定事件的默认行为。只要将 returnValue 设置为 false，就可以阻止默认行为。

```js
var link = document.getElementById("myLink");
link.onclick = function(){
    window.event.returnValue = false;
};
```
&emsp;&emsp;这个例子在onclick 事件处理程序中使用returnValue 达到了阻止链接默认行为的目的。与 DOM不同的是，在此没有办法确定事件是否能被取消。
&emsp;&emsp;相应地， cancelBubble 属性与 DOM 中的 stopPropagation()方法作用相同，都是用来停止事件冒泡的。由于 IE 不支持事件捕获，因而只能取消事件冒泡；但 stopPropagatioin()可以同时取消事件捕获和冒泡。

```js
var btn = document.getElementById("myBtn");
btn.onclick = function(){
    alert("Clicked");
    window.event.cancelBubble = true;
};
document.body.onclick = function(){
    alert("Body clicked");
};
```
### 13.3.3 跨浏览器的事件对象

&emsp;&emsp;虽然 DOM 和 IE 中的 event 对象不同，但基于它们之间的相似性依旧可以拿出跨浏览器的方案来。IE 中 event 对象的全部信息和方法 DOM 对象中都有，只不过实现方式不一样。

```js
var EventUtil = {
    addHandler: function(element, type, handler){
    //省略的代码
    },
    getEvent: function(event){
        return event ? event : window.event;
    },
    getTarget: function(event){
        return event.target || event.srcElement;
    },
    preventDefault: function(event){
        if (event.preventDefault){
            event.preventDefault();
        } else {
            event.returnValue = false;
        }
    },
    removeHandler: function(element, type, handler){
        //省略的代码
    },
    stopPropagation: function(event){
        if (event.stopPropagation){
            event.stopPropagation();
        } else {
            event.cancelBubble = true;
        }
    }
};

```

```js
btn.onclick = function(event){
    event = EventUtil.getEvent(event);
    var target = EventUtil.getTarget(event);
    EventUtil.preventDefault(event);
};
```
## 13.4 事件类型
&emsp;&emsp;“DOM3级事件”规定了以下几类事件。
-  UI（User Interface，用户界面）事件，当用户与页面上的元素交互时触发；
-  焦点事件，当元素获得或失去焦点时触发；
-  鼠标事件，当用户通过鼠标在页面上执行操作时触发；
-  滚轮事件，当使用鼠标滚轮（或类似设备）时触发；
-  文本事件，当在文档中输入文本时触发；
-  键盘事件，当用户通过键盘在页面上执行操作时触发；
-  合成事件，当为 IME（Input Method Editor，输入法编辑器）输入字符时触发；
-  变动（mutation）事件，当底层 DOM 结构发生变化时触发。
-  变动名称事件，当元素或属性名变动时触发。此类事件已经被废弃，没有任何浏览器实现它们，因此本章不做介绍。

### 13.4.1 UI 事件
&emsp;&emsp;UI 事件指的是那些不一定与用户操作有关的事件。这些事件在 DOM 规范出现之前，都是以这种或
那种形式存在的，而在 DOM 规范中保留是为了向后兼容。现有的 UI 事件如下。
 DOMActivate：表示元素已经被用户操作（通过鼠标或键盘）激活。这个事件在 DOM3 级事件中被废弃，但 Firefox 2+和 Chrome 支持它。考虑到不同浏览器实现的差异，不建议使用这个事件。

略略略~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
## 13.5 内存和性能
&emsp;&emsp;在 JavaScript 中，添加到页面上的事件处理程序数量将直接关系到页面的整体运行性能。导致这一问题的原因是多方面的。首先，每个函数都是对象，都会占用内存；内存中的对象越多，性能就越差。其次，事先指定所有事件处理程序而导致的 DOM 访问次数，会延迟整个页面的交互就绪时间。事实上，从如何利用好事件处理程序的角度出发，还是有一些方法能够提升性能的。
### 13.5.1 事件委托
&emsp;&emsp;对“事件处理程序过多”问题的解决方案就是事件委托。事件委托利用了事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。例如， click 事件会一直冒泡到 document 层次。也就是说，我们可以为整个页面指定一个 onclick 事件处理程序，而不必给每个可单击的元素分别添加事件处理程序。以下面的 HTML 代码为例。

```html
<ul id="myLinks">
    <li id="goSomewhere">Go somewhere</li>
    <li id="doSomething">Do something</li>
    <li id="sayHi">Say hi</li>
</ul>
```

```js
var list = document.getElementById("myLinks");
EventUtil.addHandler(list, "click", function(event){
    event = EventUtil.getEvent(event);
    var target = EventUtil.getTarget(event);
    switch(target.id){
        case "doSomething":
            document.title = "I changed the document's title";
        break;
        case "goSomewhere":
            location.href = "http://www.wrox.com";
        break;
        case "sayHi":
            alert("hi");
        break;
    }
});

```
&emsp;&emsp;如果可行的话，也可以考虑为 document 对象添加一个事件处理程序，用以处理页面上发生的某种特定类型的事件。这样做与采取传统的做法相比具有如下优点。
-  document 对象很快就可以访问，而且可以在页面生命周期的任何时点上为它添加事件处理程序（无需等待 DOMContentLoaded 或 load事件）。换句话说，只要可单击的元素呈现在页面上，就可以立即具备适当的功能。
-  在页面中设置事件处理程序所需的时间更少。只添加一个事件处理程序所需的 DOM 引用更少，所花的时间也更少。
-  整个页面占用的内存空间更少，能够提升整体性能。

&emsp;&emsp;最适合采用事件委托技术的事件包括 click、mousedown、mouseup、keydown、keyup 和 keypress。虽然 mouseover 和 mouseout 事件也冒泡，但要适当处理它们并不容易，而且经常需要计算元素的位置。
### 13.5.2 移除事件处理程序
&emsp;&emsp;每当将事件处理程序指定给元素时，运行中的浏览器代码与支持页面交互的 JavaScript 代码之间就会建立一个连接。这种连接越多，页面执行起来就越慢。如前所述，可以采用事件委托技术，限制建立的连接数量。另外，在不需要的时候移除事件处理程序，也是解决这个问题的一种方案。内存中留有那些过时不用的“空事件处理程序”（dangling event handler），也是造成 Web 应用程序内存与性能问题的主要原因。

&emsp;&emsp;在两种情况下，可能会造成上述问题。

&emsp;&emsp;第一种情况就是从文档中移除带有事件处理程序的元素时。这可能是通过纯粹的 DOM 操作，例如使用 removeChild()和 replaceChild()方法，但更多地是发生在使用 innerHTML 替换页面中某一部分的时候。如果带有事件处理程序的元素被 innerHTML 删除了，那么原来添加到元素中的事件处理程序极有可能无法被当作垃圾回收。来看下面的例子。

```html
<div id="myDiv">
    <input type="button" value="Click Me" id="myBtn">
</div>
<script type="text/javascript">
    var btn = document.getElementById("myBtn");
    btn.onclick = function(){
        //先执行某些操作
        document.getElementById("myDiv").innerHTML = "Processing..."; //麻烦了！
    };
</script>

```
&emsp;&emsp;这里，有一个按钮被包含在&lt;div&gt;元素中。为避免双击，单击这个按钮时就将按钮移除并替换成一条消息；这是网站设计中非常流行的一种做法。但问题在于，当按钮被从页面中移除时，它还带着一个事件处理程序呢。在&lt;div&gt;元素上设置 innerHTML 可以把按钮移走，但事件处理程序仍然与按钮保持着引用关系。有的浏览器（尤其是 IE）在这种情况下不会作出恰当地处理，它们很有可能会将对元素和对事件处理程序的引用都保存在内存中。如果你知道某个元素即将被移除，那么最好手工移除事件处理程序，如下面的例子所示。


```html
<div id="myDiv">
    <input type="button" value="Click Me" id="myBtn">
</div>
<script type="text/javascript">
    var btn = document.getElementById("myBtn");
    btn.onclick = function(){
        //先执行某些操作
        btn.onclick = null; //移除事件处理程序
        document.getElementById("myDiv").innerHTML = "Processing...";
    };
</script>

```
&emsp;&emsp;注意，在事件处理程序中删除按钮也能阻止事件冒泡。目标元素在文档中是事件冒泡的前提。
&emsp;&emsp;另一种情况，就是卸载页面的时候。毫不奇怪， IE8 及更早版本在这种情况下依然是问题最多的浏览器，尽管其他浏览器或多或少也有类似的问题。如果在页面被卸载之前没有清理干净事件处理程序，那它们就会滞留在内存中。每次加载完页面再卸载页面时（可能是在两个页面间来回切换，也可以是单击了“刷新”按钮），内存中滞留的对象数目就会增加，因为事件处理程序占用的内存并没有被释放。

&emsp;&emsp;一般来说，最好的做法是在页面卸载之前，先通过 onunload 事件处理程序移除所有事件处理程序。对这种类似撤销的操作，我们可以把它想象成：只要是通过 onload 事件处理程序添加的东西，最后都要通过 onunload 事件处理程序将它们移除。

&emsp;&emsp;不要忘了，使用 onunload 事件处理程序意味着页面不会被缓存在 bfcache 中。如果你在意这个问题，那么就只能在 IE 中通过 onunload 来移除事件处理程序了。
## 13.6 模拟事件
















