# Fabric.js介绍 第一部分

![ ](https://github.com/kangax/fabric.js/raw/master/lib/screenshot.png)

今天我开始像您介绍[Fabric](http://fabricjs.com/)，一个功能强大的Javascript库，运行在HTML5 canvas上，Fabric为画布提供了一个缺失的对象模型，以及一个SVG解析器，一个交互层，以及一整套其他必不可少的工具。它是一个完全开放源码的项目，在MIT许可，多年来做出了许多贡献。

三年前我开始开发Fabric，在发现使用原生canvas的API之后，我正在为[printio.ru](http://printio.ru/)创建一个互动设计编辑器：我的创业公司允许用户设计自己的服装。在这些日子里，我们想要的那种交互性只存在于Flash应用中。而现在，"Fabric"成为可能。

让我们来看看吧！

## 为什么要做fabric

[Canvas](http://www.whatwg.org/specs/web-apps/current-work/multipage/the-canvas-element.html)可以让我们在网络上创造出绝对惊人的图形。但它提供的API是令人失望的。如果我们只想在画布上画几条基本的形状，不会觉得有什么繁琐。但是一旦需要任何形式的互动，任何时候改变图片或绘制更复杂的形状，代码复杂度会急剧增加。

Fabric旨在解决这个问题。

原生canvas方法只允许我们触发简单的图形命令，盲目的修改canvas的位图，想画一个矩形？使用```fillRect(left, top, width, height).```，想画一条线？使用```moveTo(left, top)``` 和 ```lineTo(x, y)```的组合命令，就好像我们**用刷子画画**，上层涂上越来越多的颜料，几乎没有控制。

Fabric不是在这么低的层次上运行，而是在原生方法之上提供简单而强大的对象模型。它负责画布状态和渲染，并让我们直接使用绘制后的“对象”。

让我们来看一个简单的例子来说明这个差异。假设我们想在画布上画一个红色的矩形。以下是我们如何使用原生的```<canvas>``` API。

```js
// 有一个id是c的canvas元素
var canvasEl = document.getElementById('c');

// 获取2d位图模型
var ctx = canvasEl.getContext('2d');

// 设置填充颜色
ctx.fillStyle = 'red';

// 创建一个坐标100，190，尺寸是20，20的矩形
ctx.fillRect(100, 100, 20, 20);
```

现在使用Fabric做同样的事情：

```js
// 用原生canvas元素创建一个fabric实例
var canvas = new fabric.Canvas('c');

// 创建一个矩形对象
var rect = new fabric.Rect({
  left: 100,
  top: 100,
  fill: 'red',
  width: 20,
  height: 20
});

// 将矩形添加到canvas画布上
canvas.add(rect);
```

![ ](http://fabricjs.com/article_assets/1.png)

在这种情况下，这两个例子非常相似，大小几乎没有差别。但是，您可以看到使用canvas的方法有多么不同。使用原生方法，我们在**上下文**中操作（表示整个画布位图的对象），在Fabric中，我们操作对象，实例化它们，更改其属性，并将其添加到画布。你可以看到这些对象是Fabric中的**第一等公民**。

但渲染纯正的红色矩形就如此无聊。我们至少可以做一些有趣的事情！也许，稍稍旋转？

旋转45度，首先使用原生的canvas方法：

```js
var canvasEl = document.getElementById('c');
var ctx = canvasEl.getContext('2d');
ctx.fillStyle = 'red';

ctx.translate(100, 100);
ctx.rotate(Math.PI / 180 * 45);
ctx.fillRect(-10, -10, 20, 20);
```

使用Fabric：

```js
var canvas = new fabric.Canvas('c');

// 创建一个45度的矩形
var rect = new fabric.Rect({
  left: 100,
  top: 100,
  fill: 'red',
  width: 20,
  height: 20,
  angle: 45
});

canvas.add(rect);
```

![ ](http://fabricjs.com/article_assets/2.png)

这里发生了什么？

我们在Fabric中所做的一切都是将对象的“角度”值更改为```45```。然而使用原生的方法，事情变得更加有趣，记住我们无法对对象进行操作，相反，我们调整整个画布位图（```ctx.translate```，```ctx.rotate```）的位置和角度，以适应我们的需要。然后，我们再次绘制矩形，但记住要正确地偏移位图（-10，-10），所以它仍然呈现在100,100点。作为练习，我们不得不在旋转画布位图时将度数转换为弧度。

我相信你刚刚开始明白为什么面料存在，以及它解决了多少低级写法。

如果在某些时候，我们想将现在熟悉的红色矩形移动到画布上稍微不同的位置怎么办？我们如何在无法操作对象的情况下执行此操作？我们会在canvas位图上调用另一个```fillRect```吗？

不完全的。调用另一个```fillRect```命令实际上在画布上绘制的东西所有之上绘制矩形。还记得我前边说的用刷子画画吗？为了“移动”它，我们需要先**擦除以前绘制的内容**，然后在新的位置绘制矩形。

```js
var canvasEl = document.getElementById('c');

...
ctx.strokRect(100, 100, 20, 20);
...

// 擦除整个画布
ctx.clearRect(0, 0, canvasEl.width, canvasEl.height);
ctx.fillRect(20, 50, 20, 20);
```

我们如何用Fabric完成这个？

```js
var canvas = new fabric.Canvas('c');
...
canvas.add(rect);
...

rect.set({ left: 20, top: 50 });
canvas.renderAll();
```

![ ](http://fabricjs.com/article_assets/3.png)

注意一个非常重要的区别。使用Fabric，在尝试“修改”任何内容之前，我们不再需要擦除内容。我们仍然使用对象，只需更改其属性，然后重新绘制画布即可获得“最新图片”。

## Fabric中的对象

我们已经看到如何通过实例化```fabric.Rect```构造函数来处理矩形。当然，Fabric也涵盖了所有其他的基本形状：圆，三角形，椭圆等。所有的这些在就是Fabric“命名空间”下的：```fabric.Circle```，```fabric.Triangle```，```fabric.Ellipse```等。

Fabric提供了7种基础形状：

- [fabric.Circle](http://fabricjs.com/docs/fabric.Circle.html)
- [fabric.Ellipse](http://fabricjs.com/docs/fabric.Ellipse.html)
- [fabric.Line](http://fabricjs.com/docs/fabric.Line.html)
- [fabric.Polygon](http://fabricjs.com/docs/fabric.Polygon.html)
- [fabric.Polyline](http://fabricjs.com/docs/fabric.Polyline.html)
- [fabric.Rect](http://fabricjs.com/docs/fabric.Rect.html)
- [fabric.Triangle](http://fabricjs.com/docs/fabric.Triangle.html)

想画一个圆？只需创建一个圆形对象，并将其添加到画布。与任何其他基本形状相同：

```js
var circle = new fabric.Circle({
  radius: 20, fill: 'green', left: 100, top: 100
});
var triangle = new fabric.Triangle({
  width: 20, height: 30, fill: 'blue', left: 50, top: 50
});

canvas.add(circle, triangle);
```

![ ](http://fabricjs.com/article_assets/4.png)

这是一个绿色的圆形在100，100的位置，蓝色的三角形在50，50的位置。

### 操作对象

创建图形对象矩形，圆形或其他东西，当然只是开始。在某些时候，我们可能想修改这些对象。或许某些行为需要触发状态的变化，或进行某种动画。或者我们可能希望在某些鼠标交互中更改对象属性（颜色，不透明度，大小，位置）。

Fabric为我们处理画布渲染和状态管理。我们只需要自己修改对象。

以前的例子演示了```set```方法，以及如何从对象前一个位置调用```set({left：20，top：50})```来移动对象。以类似的方式，我们可以改变对象的任何其他属性。但这些属性是什么？

那么，正如你所期望的那样，可以改变定位：**top**，**left**。尺寸：**width**，**height**。渲染： **fill**, **opacity**, **stroke**, **strokeWidth**。缩放和旋转：**scaleX**, **scaleY**, **angle**;甚至可以翻转：**flipX**，**flipY**。歪斜：**skewX**，**skewY**。

是的，在Fabric中翻转对象非常简单，将```flip[X|Y]```设置为```true```即可。

您可以通过```get```方法读取任何这些属性，并通过```set```进行设置。我们尝试改变一些红色矩形的属性：

```js
var canvas = new fabric.Canvas('c');
...
canvas.add(rect);

rect.set('fill', 'red');
rect.set({ strokeWidth: 5, stroke: 'rgba(100,200,200,0.5)' });
rect.set('angle', 15).set('flipY', true);
```

![ ](http://fabricjs.com/article_assets/5.png)

首先，我们将“fill”值设置为“red”，使对象成为红色。下一个语句设置“strokeWidth”和“stroke”值，给出矩形为淡绿色的5px笔画。最后，我们正在改变“angle”和“flipY”的属性。注意每个3个语句中的每个语句使用的语法略有不同。

这表明```set```是一种通用的方法。你可能经常使用它，所以它被设计得尽可能的方便。

说了如何```set```的方法，那么如何获取呢？这就有了与之对应的```get```方法，h还有一些具体的```get*```，要读取对象的“width”值，可以使用```get('width')```或```getWidth()```。获取“scaleX”值使用```get('scaleX')```或```getScaleX()```，等等。对于每个“公有”对象属性(“stroke”，“strokeWidth”，“angle”等)，都可以使用getWidth或getScaleX等方法。

您可能会注意到，在早期的示例中，对象在实例化的时候，我们直接传入的配置参数，而上边的例子我们才实例化对象的时候没有传入配置，而是使用的```set```方法传递配置。这是因为它是完全一样的。您可以在创建时“配置”对象，也可以使用```set```方法:

```js
var rect = new fabric.Rect({ width: 10, height: 20, fill: '#f55', opacity: 0.7 });

// 或者这样

var rect = new fabric.Rect();
rect.set({ width: 10, height: 20, fill: '#f55', opacity: 0.7 });
```

### 默认设置

关于这一点，您可能会问，当我们创建对象而不传递任何“配置”对象时会发生什么。它还有这些属性吗？

当然是。 Fabric 中的对象总是具有一组默认的属性值。
在创建过程中参数未被传递时会被设置成默认的参数。我们可以自己试试看看：

```js
var rect = new fabric.Rect(); // 注意没有传递参数

rect.get('width'); // 0
rect.get('height'); // 0

rect.get('left'); // 0
rect.get('top'); // 0

rect.get('fill'); // rgb(0,0,0)
rect.get('stroke'); // null

rect.get('opacity'); // 1
```

我们的矩形有一组默认的属性。它位于 0,0，黑色，完全不透明，没有描边，**没有尺寸**（宽度和高度为0）。由于没有尺寸，我们无法在画布上看到它。但是给它宽度/高度的任何正值肯定会在画布的左上角显示一个黑色矩形。

![ ](http://fabricjs.com/article_assets/6.png)

### 层次和继承

Fabric对象不仅彼此独立存在。它们形成一个非常精确的层次。

大多数对象从根```fabric.Object```继承。```fabric.Object```几乎代表二维形状，位于二维canvas平面，它是一个具有left / top和width / height属性的实体，以及一些其他图形特征。我们在物体上看到的那些属性：fill，stroke，angle，opacity，flip[X|Y]等，对于从```fabric.Object```继承的所有Fabric对象都是通用的。

这个继承允许我们在```fabric.Object```上定义方法，并在所有的“类”之间共享它们。例如，如果您想在所有对象上使用```getAngleInRadians```方法，您只需在```fabric.Object.prototype```上创建它即可：

```js
fabric.Object.prototype.getAngleInRadians = function() {
  return this.get('angle') / 180 * Math.PI;
};

var rect = new fabric.Rect({ angle: 45 });
rect.getAngleInRadians(); // 0.785...

var circle = new fabric.Circle({ angle: 30, radius: 10 });
circle.getAngleInRadians(); // 0.523...

circle instanceof fabric.Circle; // true
circle instanceof fabric.Object; // true
```

您可以看到，方法立即在所有实例上可用。

虽然子类“class”继承自```fabric.Object```，但它们通常也定义自己的方法和属性。例如，```fabric.Circle```需要有“radius”属性。而```Fabric.Image```（我们稍后会看），需要使用```getElement``` / ```setElement```方法来访问/设置图像实例的真实HTML```<img>```元素。

使用原型来获取自定义渲染和行为对于高级项目来说是非常普遍的。

## Canvas

现在我们更详细地讨论了对象，让我们回到 canvas。

首先你可以看到所有的Fabric例子如果创建 canvas 对象：```new fabric.Canvas('...')```。```fabric.Canvas```作为围绕```<canvas>```元素的包装器，并负责管理该canvas上的所有Fabric对象。它需要一个元素的id，并返回一个```fabric.Canvas```的实例。

我们可以```add```对象，引用它们，或者```remove```它们：

```js
var canvas = new fabric.Canvas('c');
var rect = new fabric.Rect();

canvas.add(rect); // 添加对象

canvas.item(0); // 源于刚刚添加的第一个矩形对象
canvas.getObjects(); // 获取所有对象（只有一个矩形）

canvas.remove(rect); // 移除这个矩形
```

尽管管理对象是```fabric.Canvas```的主要目的。其实它本身也是**可配置**的，需要为整个画布设置背景颜色或图像？将所有内容剪切到某个区域？设置不同的宽度/高度？指定画布是否互动？所有这些选项（和其他）可以在```fabric.Canvas```上设置，无论是在创建时还是之后：

```js
var canvas = new fabric.Canvas('c', {
  backgroundColor: 'rgb(100,100,200)',
  selectionColor: 'blue',
  selectionLineWidth: 2
  // ...
});

// 或者

var canvas = new fabric.Canvas('c');
canvas.setBackgroundImage('http://...');
canvas.onFpsUpdate = function(){ /* ... */ };
// ...
```

### 互动

虽然我们是一个canvas元素的主题，我们来谈谈互动。Fabric的独特功能之一，在我们刚刚看到的所有的对象模型之上，是一层交互性。

存在对象模型以允许编程访问和操纵画布上的对象。但在外部，在用户层面上，有一种方式可以通过鼠标（或触摸，触摸设备）来操纵这些对象。一旦您通过```new fabric.Canvas（'...'）```初始化画布，可以选择对象，拖动它们，缩放或旋转它们，甚至组合在一起操纵一个**组合**！

![ ](http://fabricjs.com/article_assets/7.png)
![ ](http://fabricjs.com/article_assets/8.png)

如果我们希望允许用户在画布上拖动某些东西，比如一个图片，我们需要做的就是初始化画布并在其上添加一个对象。不需要额外的配置或设置。

为了控制这种交互性，我们可以在画布上使用 Fabric 的 `selection` 布尔属性，结合各个对象的 `selectable` 布尔属性来使用。

```js
var canvas = new fabric.Canvas('c');
...
canvas.selection = false; // 禁止所有选中
rect.set('selectable', false); // 只是禁止这个矩形选中
```

但是如果根本不想要这样的互动呢？如果是这样，您可以随时用```fabric.StaticCanvas```替换```fabric.Canvas```。初始化的语法是相同的;初始化时使用```StaticCanvas```而不是```Canvas```。

```js
var staticCanvas = new fabric.StaticCanvas('c');

staticCanvas.add(
  new fabric.Rect({
    width: 10, height: 20,
    left: 100, top: 100,
    fill: 'yellow',
    angle: 30
  }));
```

这将创建一个“更轻”的画布版本，没有任何事件处理逻辑。请注意，您仍然有一个完整的对象模型来使用：添加对象，删除或修改它们，以及更改任何画布配置，所有这一切仍然有效。这只是事件处理没有了。

当我们浏览自定义构建选项时，您会看到，如果```StaticCanvas```是您需要的选项，您甚至可以创建一个较轻的Fabric版本。如果您需要非交互式图表，或者在应用程序中使用过滤器的非交互式图像，这可能是一个不错的选择。

## 图像

说到图像...

在画布上添加矩形和圆圈很有趣，但为什么我们不玩某些图像？正如你现在想象的那样，Fabric 使这个很容易。我们来实例化```fabric.Image```对象并将其添加到画布中：

```html
<canvas id="c"></canvas>
<img src="my_image.png" id="my-image">
```

```js
var canvas = new fabric.Canvas('c');
var imgElement = document.getElementById('my-image');
var imgInstance = new fabric.Image(imgElement, {
  left: 100,
  top: 100,
  angle: 30,
  opacity: 0.85
});
canvas.add(imgInstance);
```

注意我们如何将图像元素传递给```fabric.Image```构造函数。这将创建一个```fabric.Image```的实例，就像文档中的图像一样。此外，我们立即将左/上值设置为100/100，角度为30，不透明度为0.85。一旦添加到画布中，图像呈现在100,100位置，30度角，并且稍微透明！不错。

![ ](http://fabricjs.com/article_assets/9.png)

现在，如果我们在文档中真的没有图像，而只是一个图像的URL呢？我们来看看如何使用```fabric.Image.fromURL```：

```js
fabric.Image.fromURL('my_image.png', function(oImg) {
  canvas.add(oImg);
});
```

看起来很简单，不是吗？只需调用具有图像URL的```fabric.Image.fromURL```，并在加载和创建图像后给它一个回调函数来调用。回调函数接收已经创建的```fabric.Image```对象作为第一个参数。此时，您可以将其添加到画布中，也可以先更改，然后添加到画布：

```js
fabric.Image.fromURL('my_image.png', function(oImg) {
  // 将其缩小，然后将其翻转，然后再将其添加到画布上
  oImg.scale(0.5).set('flipX, true);
  canvas.add(oImg);
});
```

## 路径（Paths）

我们已经看过简单的形状，然后看了图像。那么更复杂，丰富的形状和内容呢？

认识更强大的搭档：路径（Path）和组合（Groups）

Fabric 中的 paths 表示可以填充，描边和修改的形状的轮廓。Paths 由一系列命令组成，基本上模仿了从一个点到另一个点的笔。借助“move”，“line”，“curve”或“arc”等命令，path可以形成令人难以置信的复杂形状。在Paths（路径组合）团队的帮助下，可能性更大。

Fabric 中的路径与 [SVG <path>元素](http://www.w3.org/TR/SVG/paths.html#PathElement) 非常相似。它们使用相同的命令，可以从```<path>```元素创建，并将其序列化。稍后我们将更加仔细地观察序列化和SVG解析，但现在值得一提的是，您很可能很少手动创建Path实例，相反，您将使用Fabric的内置SVG解析器。但是要了解Path对象是什么，我们来尝试用手创建一个简单的对象：

```js
var canvas = new fabric.Canvas('c');
var path = new fabric.Path('M 0 0 L 200 100 L 170 200 z');
path.set({ left: 120, top: 120 });
canvas.add(path);
```

![ ](http://fabricjs.com/article_assets/10.png)

我们通过传递一串路径指令，实例化```fabric.Path```对象，虽然看起来很神秘，但实际上很容易理解。“M”代表“move”命令，并告诉笔移动到```0,0```坐标。“L”表示“line”，笔画线为```200,100```坐标。然后，另一个“L”创建一个连接```170,200```坐标的线段。最后，“z”指示绘画笔关闭当前路径并完成形状。结果，我们得到一个三角形。

由于```fabric.Path```与Fabric中的任何其他对象一样，我们还可以更改其某些属性。但是我们可以修改更多：

```js
...
var path = new fabric.Path('M 0 0 L 300 100 L 200 300 z');
...
path.set({ fill: 'red', stroke: 'green', opacity: 0.5 });
canvas.add(path);
```

![ ](http://fabricjs.com/article_assets/11.png)

出于好奇，我们来看一下稍微更复杂的path语法。你会看到为什么手动创建路径可能不是最好的主意。

```js
...
var path = new fabric.Path('M121.32,0L44.58,0C36.67,0,29.5,3.22,24.31,8.41\
c-5.19,5.19-8.41,12.37-8.41,20.28c0,15.82,12.87,28.69,28.69,28.69c0,0,4.4,\
0,7.48,0C36.66,72.78,8.4,101.04,8.4,101.04C2.98,106.45,0,113.66,0,121.32\
c0,7.66,2.98,14.87,8.4,20.29l0,0c5.42,5.42,12.62,8.4,20.28,8.4c7.66,0,14.87\
-2.98,20.29-8.4c0,0,28.26-28.25,43.66-43.66c0,3.08,0,7.48,0,7.48c0,15.82,\
12.87,28.69,28.69,28.69c7.66,0,14.87-2.99,20.29-8.4c5.42-5.42,8.4-12.62,8.4\
-20.28l0-76.74c0-7.66-2.98-14.87-8.4-20.29C136.19,2.98,128.98,0,121.32,0z');

canvas.add(path.set({ left: 100, top: 200 }));
```

看一下这里发生了什么？

那么“M”仍然代表“move”的命令，所以笔在“121.32,0”点开始绘画。然后有“L”命令使其为“44.58,0”。到现在为止还可以理解。下一步是什么？ “C”指令，代表“cubic bezier”（贝塞尔曲线）。它使笔从当前点绘制贝塞尔曲线到“36.67,0”，它以“29.5,3.22”为起点的控制点，“24.31,8.41”为终点的控制点。这整个事情之后是十几个其他的“cubic bezier”命令，最终创建一个漂亮的箭头形状。

![ ](http://fabricjs.com/article_assets/12.png)

不过你可能永远不会使用这样复杂的命令，相反，您可能需要使用像```fabric.loadSVGFromString```或```fabric.loadSVGFromURL```方法来加载整个SVG文件,然后让Fabric的SVG解析器完成对所有SVG元素的遍历和创建相应的Path对象的工作。

谈到整个SVG文件，而Fabric的路径通常表示SVG```<path>```元素，SVG文档中通常存在的路径集合表示为Group(```fabric.Group```实例)。你可以想像，Group(组合)只不过是一组Path实例和其他对象。而且由于```fabric.Group```从```fabric.Object```继承，它可以像任何其他对象一样添加到画布中，并以相同的方式进行操作。

就像Path一样，你可能不会直接的使用它。但是，如果您在解析SVG文档之后偶然发现了一个问题，那么您将确切地知道它是什么以及它的目的是什么。

## 后记

我们只触及了Fabric的表面。你现在可以很容易地创建任何一个简单的形状，复杂的形状，图像;将它们添加到画布中，并以任何你想要的方式进行修改：位置、尺寸、角度、颜色、笔画、不透明度等。

在本系列的下一部分，我们将看看组合，动画，文本，SVG解析，渲染，序列化，事件，图像，滤镜等。

与此同时，你可以随意查看带注释的[演示](http://fabricjs.com/demos/)或[基准](http://fabricjs.com/benchmarks/)，加入[Google小组](https://groups.google.com/forum/?fromgroups#!forum/fabricjs)或[其他地方](http://stackoverflow.com/questions/tagged/fabricjs)的讨论，或者直接访问[docs](http://fabricjs.com/docs/)、[wiki](https://github.com/kangax/fabric.js/wiki)和[源代码](https://github.com/kangax/fabric.js)。

做一些有趣的实验！我希望你喜欢这次旅行。