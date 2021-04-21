# 浏览器的回流与重绘 (Reflow & Repaint)

在讨论回流与重绘之前，我们要知道:
	1. 浏览器使用流式布局模型 (Flow Based Layout)。
	2. 浏览器会把HTML解析成DOM，把CSS解析成CSSOM，DOM和CSSOM合并就产生了Render Tree。
	3. 有了RenderTree，我们就知道了所有节点的样式，然后计算他们在页面上的大小和位置，最后把节点绘制到页面上。
	4. 由于浏览器使用流式布局，对Render Tree的计算通常只需要遍历一次就可以完成，但table及其内部元素除外，他们可能需要多次计算，通常要花3倍于同等元素的时间，这也是为什么要避免使用table布局的原因之一。


#### 一句话：回流必将引起重绘，重绘不一定会引起回流。

## 回流 (Reflow)
当Render Tree中部分或全部元素的尺寸、结构、或某些属性发生改变时，浏览器重新渲染部分或全部文档的过程称为回流。
会导致回流的操作：
	* 页面首次渲染
	* 浏览器窗口大小发生改变
	* 元素尺寸或位置发生改变
	* 元素内容变化（文字数量或图片大小等等）
	* 元素字体大小变化
	* 添加或者删除可见的DOM元素
	* 激活CSS伪类（例如：:hover）
	* 查询某些属性或调用某些方法
	* 计算 offsetWidth 和 offsetHeight 属性（Calculating offsetWidth and offsetHeight）
	* 设置 style 属性的值 （Setting a property of the style attribute）
	* 操作 class 属性（Manipulating the class attribute）


一些常用且会导致回流的属性和方法：
	* clientWidth、clientHeight、clientTop、clientLeft
	* offsetWidth、offsetHeight、offsetTop、offsetLeft
	* scrollWidth、scrollHeight、scrollTop、scrollLeft
	* scrollIntoView()、scrollIntoViewIfNeeded()
	* getComputedStyle()
	* getBoundingClientRect() // 获取某个元素相对于视窗的位置集合
	* scrollTo()
  
 ## 重绘 (Repaint)
当页面中元素样式的改变并不影响它在文档流中的位置时（例如：color、background-color、visibility等），浏览器会将新样式赋予给元素并重新绘制它，这个过程称为重绘。性能影响
回流比重绘的代价要更高。
有时即使仅仅回流一个单一的元素，它的父元素以及任何跟随它的元素也会产生回流。
现代浏览器会对频繁的回流或重绘操作进行优化：
浏览器会维护一个队列，把所有引起回流和重绘的操作放入队列中，如果队列中的任务数量或者时间间隔达到一个阈值的，浏览器就会将队列清空，进行一次批处理，这样可以把多次回流和重绘变成一次。
当你访问以下属性或方法时，浏览器会立刻清空队列：
	* clientWidth、clientHeight、clientTop、clientLeft  // clientWidth = width+左右padding
	* offsetWidth、offsetHeight、offsetTop、offsetLeft    //offsetWidth = width + 左右padding + 左右boder 
	* scrollWidth、scrollHeight、scrollTop、scrollLeft // 获取指定标签内容层的真实宽度（可视区域宽度+被隐藏区域宽度
	* width、height
	* getComputedStyle()
	* getBoundingClientRect()


因为队列中可能会有影响到这些属性或方法返回值的操作，即使你希望获取的信息与队列中操作引发的改变无关，浏览器也会强行清空队列，确保你拿到的值是最精确的。

## 如何避免
#### CSS
	* 避免使用table布局。（ 避免使用 table布局。可能您需要其它些避免使用table的理由，在布局完全建立之前，table经常需要多个关口，因为table是个和罕见的可以影响在它们之前已经进入的DOM元素的显示的元素。想象一下，因为表格最后一个单元格的内容过宽而导致纵列大小完全改变。这就是为什么所有的浏览器都逐步地不支持table表格的渲染（感谢Bill Scott提供）。然而有另外一个原因为什么表格布局时很糟糕的主意，根据 Mozilla，即使一些小的变化将导致表格(table)中的所有其他节点回流。）
	* 尽可能在DOM树的最末端改变class。（ 回流可以自上而下，或自下而上的回流的信息传递给周围的节点。回流是不可避免的，但可以减少其影响。尽可能在DOM树的里面改变class，可以限制了回流的范围，使其影响尽可能少的节点。例如，你应该避免通过改变对包装元素类去影响子节点的显示。面向对象的CSS始终尝试获得它们影响的类对象（DOM节点或节点），但在这种情况下，它已尽可能的减少了回流的影响，增加性能优势。）
	* 避免设置多层内联样式。（ 我们都知道与DOM交互很慢。我们尝试在一种无形的DOM树片段组进行更改，然后整个改变应用到DOM上时仅导致了一个回流。同样，通过style属性设置样式导致回流。避免设置多级内联样式，因为每个都会造成回流，样式应该合并在一个外部类，这样当该元素的class属性可被操控时仅会产生一个reflow。）
	* 将动画效果应用到position属性为absolute或fixed的元素上。（ 动画效果应用到position属性为absolute或fixed的元素上，它们不影响其他元素的布局，所它他们只会导致重新绘制，而不是一个完整回流。这样消耗会更低。）
	* 避免使用CSS表达式（例如：calc()）（在css里写js表达式 这项规则较过时，但确实是个好的主意。主要的原因，这些表现是如此昂贵，是因为他们每次重新计算文档，或部分文档、回流。正如我们从所有的很多事情看到的：引发回流，它可以每秒产生成千上万次。当心！）。

#### JavaScript
	* 避免频繁操作样式，最好一次性重写style属性，或者将样式列表定义为class并一次性更改class属性。
	* 避免频繁操作DOM，创建一个documentFragment，在它上面应用所有DOM操作，最后再把它添加到文档中。
	* 也可以先为元素设置display: none，操作结束后再把它显示出来。因为在display属性为none的元素上进行的DOM操作不会引发回流和重绘。
	* 避免频繁读取会引发回流/重绘的属性，如果确实需要多次使用，就用一个变量缓存起来。
	* 对具有复杂动画的元素使用绝对定位，使它脱离文档流，否则会引起父元素及后续元素频繁回流。


#### 补充说明

documentFragment是一个保存多个element的容器对象（保存在内存）当更新其中的一个或者多个element时，页面不会更新。只有当documentFragment容器中保存的所有element更新后再将其插入到页面中才能更新页面。
documentFragment用来批量更新
列如将ul里面的li取出放到documentFragment,更新完毕后再将其插入到ul,一共有以下四步骤：
	1. 创建documentFragment对象fragment
	2. 取出ul中的所有子节点并保存到fragment
	3. 更新fragment中的所有节点（li的内容）
	4. 将fragment插入到ul

原文地址：http://www.stubbornella.org/content/2009/03/27/reflows-repaints-css-performance-making-your-javascript-slow/
