# Fabric.js介绍 第四部分

我们已经在前几个系列涵盖了很多话题，从基础对象操作到动画，事件，滤镜，组合和子类。但还有几件非常有趣和有用的事情要讨论！

## 自由绘画

如果说有什么功能能让```<canvas>```在眼前一亮，那一定是它能够很好的支持自由绘图！由于画布只是一个2D位图， Fabric为我们提供了一张纸：可以自由绘画，而且非常自然。

只需将Fabric canvas的```isDrawingMode```属性设置为```true```即可实现自由绘制模式。这样画布上的点击和移动就会被立刻解释为铅笔或刷子。

当```isDrawingMode```为```true```时，您可以随意在画布上多次绘制，但是一旦你在画布上做任何动作，然后触发“mouseup”事件，Fabric就会触发"path:created" 事件，并且将刚刚的绘画转变为```fabric.Path```的实例，

如果在任何时刻，将```isDrawingMode```设置为```false```，你创建的路径对象会仍然存在于画布上。而且由于它们是```fabric.Path```对象，所以可以以任何方式修改它们： 移动，旋转，缩放等。

还有两个属性可以定制自由绘图： ```freeDrawingBrush.color``` and ```freeDrawingBrush.width```，两者都可以通过```freeDrawingBrush```实例在Fabric画布实例上使用。```freeDrawingBrush.color```可以是任何常规的颜色值，代表画笔的颜色。```freeDrawingBrush.width```是一个像素，代表画笔的宽度。

![ ](http://fabricjs.com/article_assets/4_1.png)

在不久的将来，我们计划添加更多的自由绘图选项：各种版本的画笔（如喷雾式或粉笔式）。还可以自定义画笔模式，还有一个可以自己扩展的选项，类似于Fabric图像滤镜。

## 自定义

一件有趣的事情是如何去自定义Fabric，您可以在```canvas```或```canvas```对象上调整数十种各样的参数，以使事情按照所需的方式运行。我们来看看其中的一些。

### 锁定对象

画布上的每个对象都可以以几种方式锁定。“lockMovementX”，“lockMovementY”，“lockRotation”，“lockScalingX”，“lockScalingY”是锁定相应对象动作的属性。所以将某个对象的```lockMovementX```属性设置为```true```会阻止对象被水平移动，此时你仍然可以垂直移动它。类似的，```lockRotation```可以阻止旋转，```lockScalingX``` / ```lockScalingY``` 可以阻止水平或者垂直缩放。所有这些都是可以叠加的，你可以将它们组合在一起使用。

### 改变边框和角

您可以通过“hasControls”和“hasBorders”属性来控制对象的边框和角的可见性。只是将它们设置为```false```。

```js
object.hasBorders = false;
```

![ ](http://fabricjs.com/article_assets/4_2.png)

```js
object.hasControls = false;
```

![ ](http://fabricjs.com/article_assets/4_3.png)

您还可以通过某些自定义属性来调整它的外观：“cornerDashArray”，“borderDashArray”，“borderColor”，“transparentCorners”“cornerColor”，“cornerStrokeColor”，“cornerStyle”，“selectionBackgroundColor”，“padding”和“cornerSize”。

```js
object.set({
  borderColor: 'red',
  cornerColor: 'green',
  cornerSize: 6
});
```

![ ](http://fabricjs.com/article_assets/4_4.png)

```js
  object.set({
    transparentCorners: false,
    cornerColor: 'blue',
    cornerStrokeColor: 'red',
    borderColor: 'red',
    cornerSize: 12,
    padding: 10,
    cornerStyle: 'circle',
    borderDashArray: [3, 3]
  });
  
```

![ ](http://fabricjs.com/article_assets/4_4b.png)

### 禁用选择

您可以通过将canvas的“selection”属性设置为```false```来禁用画布上的对象选择，这可以防止在画布上选中任何对象。如果只需要使某些对象不可选，则可以更改某个对象的“selectable”属性，只是将其设置为```false```，并且对象失去其交互性。

### 自定义选择

现在，如果你不想禁用选择，而是要改变它的外观呢？没问题。

canvas画布上有4个属性来控制它的表现： "selectionColor", "selectionBorderColor", "selectionLikeWidth", and "selectionDashArray"。这些都非常的语义化，让我们来看一个例子：

```js
canvas.add(new fabric.Circle({ radius: 30, fill: '#f55', top: 100, left: 100 }));

canvas.selectionColor = 'rgba(0,255,0,0.3)';
canvas.selectionBorderColor = 'red';
canvas.selectionLineWidth = 5;
```

![ ](http://fabricjs.com/article_assets/4_8.png)

最后一个属性“selectionDashArray”可不是那么简单，他允许我们使用虚线作为选择线。定义虚线模式的方式是通过数组指定间隔。因此，要创建一个模式，其中有一个大点的数字，后面跟着一个小一点的数字来代表虚线的间隔，我们可以使用```[10，5]```，作为“selectionDashArray”。这将画出10px长的线，然后跳过5px，再画一个10px长度的线，以此类推。如果我们使用```[2, 4, 6]```的数组，这样将通过绘制2px长的线，然后跳过4px，然后绘制6px线，然后跳过2px，然后绘制4px线，然后跳过6px，以此类推。例如，这就是```[5,10]```规则的样子：

![ ](http://fabricjs.com/article_assets/4_9.png)

### 虚线描边

与canvas上的“selectionDashArray”类似，所有Fabric对象都有“strokeDashArray”属性，负责在对象上执行虚线的描边操作。

```js
var rect = new fabric.Rect({
  fill: '#06538e',
  width: 125,
  height: 125,
  stroke: 'red',
  strokeDashArray: [5, 5]
});
canvas.add(rect);
```

![ ](http://fabricjs.com/article_assets/4_13.png)

### 可点击区域

如你所知，所有的Fabric对象都有边界边框，用于拖动、旋转、缩放。您可能已经注意到，当用于控制的角和边框存在时，即使在单击没有绘制内容的对象边框内的空白时也可以拖动该对象。

看看这个图像：

![ ](http://fabricjs.com/article_assets/4_14.png)

默认情况下，画布上的所有Fabric对象都可以通过边框拖动。如果你想要不同的行为，例如根据Fabric对象的实际内容进行点击或拖拽，您可以在对象上使用“perPixelTargetFind”属性。只需将其设置为```true```即可获得所需的行为。

### 旋转控件

由于版本1.0 Fabric在默认情况下使用默认UI，对象不能同时缩放和旋转。作为代替的是每个对象都有一个单独的旋转控件。该控件的相应属性为“hasRotatingPoint”，默认是```true```，将其设置为```false```可以禁用旋转控制。您也可以通过“rotatePointOffset”数字属性自定义其相对于对象的偏移量。

![ ](http://fabricjs.com/article_assets/4_10.png)

### Fabric对象变形

自**1.0**版以来，Fabric中还提供了许多其他与转换相关的属性，其中一个是画布实例上的“uniScaleTransform”。默认为```false```，可用于启用对象的非均匀缩放;换句话说，它允许在通过角拖动时改变对象的比例。

![ ](http://fabricjs.com/article_assets/4_15.png)

有“centeredScaling”和“centeredRotation”属性（在v1.3.4之前是一个属性“centerTransform”）。他们指定对象的中心是否应该用作转换的起点。当它们都设置为```true```，对象总是从中心缩放/旋转时，它会和**1.0**之前的行为相同。由于**1.0**变形原点是动态的，这样可以在缩放对象时进行更精细的控制。

最后一对新属性是“originX”和“originY”。默认设置为“left”和“top”，它们允许以编程的方式更改变形的原点。当你拖动对象的角时，就是这些属性正在动态变化。

那么什么时候需要我们手动更改它们？举个例子，当我们处理文本对象时，当您动态的改变文本并且文本框尺寸变大，“originX”和“originY”指出了增长的位置。所以如果你需要文本对象居中时，你可以将originX设置为“center”。如果让文本移到右边，你需要将“originX”设置为“right”等等。这种行为类似于CSS中的“position：absolute”。

### 画布的背景和叠加

从第一部分可以记得，您可以指定一个颜色来填充整个画布背景。只需将任意常规颜色值设置为canvas的“backgroundColor”属性。

```js
canvas.add(new fabric.Circle({ radius: 30, fill: '#f55', top: 100, left: 100 }));
canvas.backgroundColor = 'rgba(0,0,255,0.3)';
canvas.renderAll();
```

![ ](http://fabricjs.com/article_assets/4_5.png)

你可以进一步去分配图像作为背景。您需要使用```setBackgroundImage```方法，传递url和一个可选的完成回调。

```js
canvas.add(new fabric.Circle({ radius: 30, fill: '#f55', top: 100, left: 100 }));
canvas.setBackgroundImage('../assets/pug.jpg', canvas.renderAll.bind(canvas));
```

![ ](http://fabricjs.com/article_assets/4_6.png)

最后，您还可以设置叠加图像，在这种情况下，它将始终显示在画布上呈现的任何对象之上。只需使用setOverlayImage，传递url和一个可选的完成回调。

```js
canvas.add(new fabric.Circle({ radius: 30, fill: '#f55', top: 100, left: 100 }));
canvas.setOverlayImage('../assets/jail_cell_bars.png', canvas.renderAll.bind(canvas));
```

![ ](http://fabricjs.com/article_assets/4_7.png)

## Node.js上的Fabric

Fabric的独特之处在于它不仅可以在客户端，浏览器中也可以在服务器上工作！当您要从客户端发送数据并在服务器上直接创建该数据的映像时，这可能很有用。或者如果您为了速度，方便性或其他原因，只是想从控制台使用Fabric API。

让我们来看看如何设置在Node环境下运行Fabric。

首先，您需要安装[Node.js](http://nodejs.org/)，如果还没有。有几种方法来安装Node，具体取决于你的操作系统。您可以按照[这些说明](http://howtonode.org/how-to-install-nodejs)或[这些说明](https://github.com/joyent/node/wiki/Installation)。

Node安装成功之后，我们需要安装[node-canvas](https://github.com/LearnBoost/node-canvas)库，node-canvas是NodeJS的Canvas实现。它依赖[Cairo](http://cairographics.org/)- 可以在Mac，Linux或Windows上运行的2D图形库。node-canvas具有专门的[安装说明](https://github.com/LearnBoost/node-canvas/wiki)，具体取决于您的操作系统。

由于Fabric运行在Node上，它作为一个NPM包。所以下一步是安装NPM。您可以在其[github repo](https://github.com/isaacs/npm)中找到安装说明，不过一般情况下当您安装Node的时候会自动安装了NPM。

最后一步是安装[Fabric](https://npmjs.org/package/fabric)包，使用NPM。这通过运行```npm install fabric```（或```npm install -g fabric```来全局安装包）完成。

我们现在运行Node控制台，应该就使用node-canvas和Fabric了：

```bash
> node
...
> typeof require('canvas'); // "function"
> typeof require('fabric'); // "object"
```

现在一切都准备好了，我们可以尝试一个简单的“helloworld”的测试。让我们创建一个helloworld.js文件：

```js
var fs = require('fs'),
    fabric = require('fabric').fabric,
    out = fs.createWriteStream(__dirname + '/helloworld.png');

var canvas = fabric.createCanvasForNode(200, 200);
var text = new fabric.Text('Hello world', {
  left: 100,
  top: 100,
  fill: '#f55',
  angle: 15
});
canvas.add(text);

var stream = canvas.createPNGStream();
stream.on('data', function(chunk) {
  out.write(chunk);
});
```

然后运行它作为```node helloworld.js```，打开helloworld.png：

![ ](http://fabricjs.com/article_assets/4_11.png)

那么这里发生了什么？我们来看看这段代码的重要部分。

首先，我们引入了Fabric```fabric = require('fabric').fabric```，然后我们用```fabric.createCanvasForNode()```替代了平时使用的```new fabric.Canvas()```来创建了Fabric的canvas画布，此方法将width和height作为参数，并创建该大小的画布（在这种不传参的情况下为200x200）。

然后有熟悉的对象创建（```new fabric.Text（）```）和添加到画布（```canvas.add（text）```）。

到目前做的一切都只是创建了一个Fabric画布，然后渲染了一个text文本，现在，如何创建在画布上呈现的任何图像？在canvas实例上使用可用的```createPNGStream```方法。```createPNGStream```返回Node的[流](http://nodejs.org/api/stream.html)，然后可以使用```on（'data'）```输出到图像文件中，并将其写入与图像文件（```fs.createWriteStream（）```）对应的流中。

```fabric.createCanvasForNode```和```canvas.createPNGStream```几乎是Node特有的唯一2种方法，其他一切都是一样的。您仍然可以按照通常的方式创建对象，将它们添加到画布上，修改，渲染等。值得一提的是，当您通过```createCanvasForNode```创建画布时，它将使用```nodeCanvas```属性进行扩展，该属性是对原始节点画布实例的引用。

### Node服务器与Fabric

例如，让我们创建一个简单的Node服务器，它将使用JSON格式的Fabric数据收听传入的请求，并输出该数据的图像。整个脚本只有25行长！

```js
var fabric = require('fabric').fabric,
    http = require('http'),
    url = require('url'),
    PORT = 8124;

var server = http.createServer(function (request, response) {
  var params = url.parse(request.url, true);
  var canvas = fabric.createCanvasForNode(200, 200);

  response.writeHead(200, { 'Content-Type': 'image/png' });

  canvas.loadFromJSON(params.query.data, function() {
    canvas.renderAll();

    var stream = canvas.createPNGStream();
    stream.on('data', function(chunk) {
      response.write(chunk);
    });
    stream.on('end', function() {
      response.end();
    });
  });
});

server.listen(PORT);
```

此片段中的大部分代码应该已经很熟悉了。它的要点在于服务器响应。我们正在创建Fabric画布，将JSON数据加载到其中，渲染它，并将最终结果作为服务器响应进行流式传输。

要测试它，我们来获取一个绿色，并且稍微旋转的矩形：

```
{"objects":[{"type":"rect","left":103.85,"top":98.85,"width":50,"height":50,"fill":"#9ae759","overlayFill":null,"stroke":null,"strokeWidth":1,"strokeDashArray":null,"scaleX":1.39,"scaleY":1.39,"angle":30,"flipX":false,"flipY":false,"opacity":0.8,"selectable":true,"hasControls":true,"hasBorders":true,"hasRotatingPoint":false,"transparentCorners":true,"perPixelTargetFind":false,"rx":0,"ry":0}],"background":"rgba(0, 0, 0, 0)"}
```

URL编码后：

```
%7B"objects"%3A%5B%7B"type"%3A"rect"%2C"left"%3A103.85%2C"top"%3A98.85%2C"width"%3A50%2C"height"%3A50%2C"fill"%3A"%239ae759"%2C"overlayFill"%3Anull%2C"stroke"%3Anull%2C"strokeWidth"%3A1%2C"strokeDashArray"%3Anull%2C"scaleX"%3A1.39%2C"scaleY"%3A1.39%2C"angle"%3A30%2C"flipX"%3Afalse%2C"flipY"%3Afalse%2C"opacity"%3A0.8%2C"selectable"%3Atrue%2C"hasControls"%3Atrue%2C"hasBorders"%3Atrue%2C"hasRotatingPoint"%3Afalse%2C"transparentCorners"%3Atrue%2C"perPixelTargetFind"%3Afalse%2C"rx"%3A0%2C"ry"%3A0%7D%5D%2C"background"%3A"rgba(0%2C%200%2C%200%2C%200)"%7D
```

并通过“data”查询参数传递给服务器。立即回应“image / png”的Content-type，如下所示：

正如你所看到的，在服务器上使用Fabric是非常简单直接的。随意试验这个片段。可以从URL参数中更改画布尺寸，或者在返回图像作为响应之前修改客户端数据。

### 在Node环境自定义Fabric的字体

在我们可以在Fabric中使用自定义字体之前，我们需要先加载它们。在浏览器（客户端）中，加载字体的最常用方法是使用[CSS3 @font-face](http://www.w3.org/TR/css3-fonts/#font-face-rule)规则。 在Node（服务器端）的Fabric中，我们可以使用node-canvas的**Font API**加载字体。

下面的示例演示如何加载和使用自定义字体。将其保存到customfont.js并确保字体文件的路径正确。在这个例子中，我们使用[Ubuntu](http://www.google.com/fonts/specimen/Ubuntu)作为自定义字体。

```js
var fs = require('fs'),
    fabric = require('fabric').fabric,
    canvas = fabric.createCanvasForNode(300, 250);

var font = new canvas.Font('Ubuntu', __dirname + '/fonts/Ubuntu-Regular.ttf');
font.addFace(__dirname + '/fonts/Ubuntu-Bold.ttf', 'bold');
font.addFace(__dirname + '/fonts/Ubuntu-Italic.ttf', 'normal', 'italic');
font.addFace(__dirname + '/fonts/Ubuntu-BoldItalic.ttf', 'bold', 'italic');

canvas.contextContainer.addFont(font);  // 使用 createPNGStream 或者 createJPEGStream 的时候
canvas.contextTop.addFont(font);        // 使用 toDataURL 或者 toDataURLWithMultiplier的时候

var text = new fabric.Text('regular', {
    left: 150,
    top: 50,
    fontFamily: 'Ubuntu'
});
canvas.add(text);

text = new fabric.Text('bold', {
    left: 150,
    top: 100,
    fontFamily: 'Ubuntu',
    fontWeight: 'bold'
});
canvas.add(text);

text = new fabric.Text('italic', {
    left: 150,
    top: 150,
    fontFamily: 'Ubuntu',
    fontStyle: 'italic'
});
canvas.add(text);

text = new fabric.Text('bold italic', {
    left: 150,
    top: 200,
    fontFamily: 'Ubuntu',
    fontWeight: 'bold',
    fontStyle: 'italic'
});
canvas.add(text);

var out = fs.createWriteStream(__dirname + '/customfont.png');
var stream = canvas.createPNGStream();
stream.on('data', function(chunk) {
    out.write(chunk);
});
```

运行```node customfont.js```创建一个图像（customfont.png），如下所示：

![ ](http://fabricjs.com/article_assets/4_16.png)

让我们仔细看看发生了什么。首先，通过将字体名称和路径（作为常规字体文件）作为参数传递，创建一个字体对象```new canvas.Font（）```。然后通过将字体path，weight和style作为参数传递，使用```font.addFace（）```添加其他字体。最后，使用```canvas.contextContainer.addFont（）```（当使用**createPNGStream**或**createJPEGStream**时）或```canvas.contextTop.addFont（）```（使用**toDataURL**或**toDataURLWithMultiplier**时）可以将字体添加到所需的上下文中。

现在我们可以通过将```fabric.Text```对象的**fontFamily**属性设置为字体名称来使用我们的字体。结合**fontWeight**和**fontStyle**属性，我们可以应用我们添加的字体。有关这些属性的更多信息，请参阅第2部分。请注意，该示例显示了如何在创建新文本对象时使用自定义字体，但这也适用于通过JSON加载的文本对象。

这就到了Fabric的4部分系列的结尾。我希望你现在拥有足够的知识，创造有趣，酷，有用，有趣，具有挑战性，令人兴奋的东西！