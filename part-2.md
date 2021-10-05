# Fabric.js介绍 第二部分

在本系列的第一部分，我们只开始熟悉Fabric.js。我们研究了Fabric在其对象模型和对象层次结构中使用Fabric的各种实例：简单的形状，图像和复杂的路径。我们还学习了如何使用画布上的Fabric对象执行简单的操作。

既然基础知识都差不多了，让我们开始学习一些有趣的东西吧！

## 动画（animate）

没有听说过哪一个强大的canvas库是没有动画功能的，当然Fabric也不例外。既然有这样一个强大的对象模型和图形功能，那么没有内置动画简直就是一种耻辱。

改变任何对象的属性是多么容易？我们调用```set```方法，传递相应的值：

```js
rect.set('angle', 45);
```

那么，设置动画对象就是一样的简单。每个Fabric对象都有```animate```方法，它可以让一级对象执行动画。

```js
rect.animate('angle', 45, {
  onChange: canvas.renderAll.bind(canvas)
});
```

第一个参数是动画的属性，第二个参数是动画的最终位置，如果矩形为-15°角，我们传递45，这个矩形将从-15°到45°执行动画。第三个参数是一个可选的对象，指定动画的细节：持续时间，回调，动效等。

```animate```的一个方便之处在于它还支持相对值。例如，如果要将对象向左移动100px，则可以这样做：

```js
rect.animate('left', '+=100', { onChange: canvas.renderAll.bind(canvas) });
```

同样的，逆时针旋转5度，可以像这样完成：

```js
rect.animate('angle', '-=5', { onChange: canvas.renderAll.bind(canvas) });
```

你可能会想知道为什么我们总是在那里指定“onChange”回调。不是说第三个参数是可选的吗？是为了每个动画帧的渲染都能让我们看到实际的动画效果！您可以看到，当我们调用```animate```方法时，它只会随着时间的推移赋予属性值，遵循特定的算法(例如动效)。所以```rect.animate（'angle'，45）```会改变对象的角度，但不会在每次角度变化后重新渲染画布屏幕。我们显然需要重新渲染才能看到实际的动画。

记住，在画布的下面有整个对象模型。对象有自己的属性和关系，而canvas只负责将它们的存在投射到外部世界。

每次更改之后，```animate```处于性能原因，不会自动重新绘制画布。毕竟，我们可以在画布上有数以千计的动画对象，如果每个对象都尝试重新渲染屏幕，这是非常不妥的。在许多对象的情况下，您可以使用像```requestAnimationFrame```（或其他基于定时器的）循环来独立地渲染画布，而不需要为每个对象调用```renderAll```。但大多数情况下，您可能需要将```canvas.renderAll```显式指定为“onChange”的回调。

那么我们可以通过哪些其他选项来动画？

- **from** 允许指定动画属性的起始值（如果我们不希望使用当前值）。
- **duration** 默认为500ms。可以用来改变动画的持续时间。
- **onComplete** 动画结束之后的回调。
- **easing** 动效函数。

除了easing，所有这些参数都是见文知意的。我们来看看吧。

默认情况下，动画使用“easeInSine”动效执行。如果这不是你需要的，那么在```fabric.util.ease```下有一大堆动效的选项。例如，如果我们想以一种有弹性的方式将对象移到左边：

```js
rect.animate('left', 500, {
  onChange: canvas.renderAll.bind(canvas),
  duration: 1000,
  easing: fabric.util.ease.easeOutBounce
});
```

注意使用了```fabric.util.ease.easeOutBounce```作为easing的值，其他值得注意的还包括```easeInCubic```，```easeOutCubic```，```easeInElastic```，```easeOutElastic```，```easeInBounce```和```easeOutExpo```。

所以这覆盖了Fabric的动画部分。只是给你一些可能的想法：使对象的angle改变来形成旋转动画，使对象的top/left改变来形成移动的动画，使对象的width/height改变来形成大小缩放的动画，等等。

## 图像滤镜（filters）

在第一节中我们学习了在Fabric中如何使用图像：```fabric.Image```构造函数，它接受图像元素。还有```fabric.Image.fromURL```方法，可以创建URL字符串的图像实例。并且这些图像可以像其他对象一样投射到canvas画布上。

但是，像图像这种有趣的东西，对他们应用图像滤镜，效果会更加炫酷！

Fabric在默认情况下提供了很少的滤镜，无论是否支持WEBGL的浏览器，都可以使用，也很容易去定义。一些你可能非常熟悉的内置的滤镜：去除白色背景、灰度滤镜、反色或亮度调色。其他的则可能不太受欢迎： colormatrix，sepia，noise。

那么我们怎么在Fabric中使用滤镜呢？```fabric.Image```的每个实例都有“filters”属性，它是一个简单的滤镜数组。该数组中的每个滤镜都是一个Fabric滤镜的一个实例，或您自己的自定义滤镜的一个实例。

让我们创建一个灰度（Grayscale）图像：

```js
fabric.Image.fromURL('pug.jpg', function(img) {
  // 添加滤镜
  img.filters.push(new fabric.Image.filters.Grayscale());
  // 图片加载完成之后，应用滤镜效果
  img.applyFilters();
  // 将处理后的图片添加到canvas画布上
  canvas.add(img);
});
```

![ ](http://fabricjs.com/article_assets/2_1.png)

图像的色偏（Sepia）版本怎么样？

```js
fabric.Image.fromURL('pug.jpg', function(img) {
  img.filters.push(new fabric.Image.filters.Sepia());
  img.applyFilters();
  canvas.add(img);
});
```

![ ](http://fabricjs.com/article_assets/2_2.png)

由于“filters”属性是一个数组，我们可以用数组方法执行任何所需的操作：移除滤镜（pop，splice，shift），添加滤镜（push，unshift，splice），甚至可以组合多个滤镜。当我们调用```applyFilters```时，“filters”数组中存在的任何滤镜将逐个应用，所以让我们尝试创建一个既色偏又明亮（Brightness）的图像。

```js
fabric.Image.fromURL('pug.jpg', function(img) {
  img.filters.push(
    new fabric.Image.filters.Sepia(),
    new fabric.Image.filters.Brightness({ brightness: 100 }));

  img.applyFilters();
  canvas.add(img);
});
```

![ ](http://fabricjs.com/article_assets/2_3.png)

请注意，我们还将```{brightness：100}```对象传递给Brightness滤镜。这是因为有些滤镜不可配置，例如，灰度，反转，棕褐色（e.g. grayscale, invert, sepia），而其他过滤器可以配置。对于Brightness滤镜，能够配置亮度级别（-1 - 1 即： 全黑色 - 全白色）。对于噪声（noise）滤镜，能配置的是噪声值（0-1000）。对于“删除颜色”滤镜，能配置的是阈值和距离值。等等。

那么现在，您已经熟悉了Fabric滤镜，现在是时候开箱并创建自己的滤镜了！

```js
fabric.Image.filters.Redify = fabric.util.createClass({

  type: 'Redify',

  /**
   * 用于编程的片段源码
   */
  fragmentSource: 'precision highp float;\n' +
    'uniform sampler2D uTexture;\n' +
    'varying vec2 vTexCoord;\n' +
    'void main() {\n' +
      'vec4 color = texture2D(uTexture, vTexCoord);\n' +
      'color.g = 0;\n' +
      'color.b = 0;\n' +
      'gl_FragColor = color;\n' +
    '}',

  applyTo2d: function(options) {
    var imageData = options.imageData,
        data = imageData.data, i, len = data.length;

    for (i = 0; i < len; i += 4) {
      data[i + 1] = 0;
      data[i + 2] = 0;
    }

  }
});

fabric.Image.filters.Brightness.fromObject = fabric.Image.filters.BaseFilter.fromObject;
```

![ ](http://fabricjs.com/article_assets/2_4.png)

深入研究这段代码，主要的操作是在循环中进行。我们把绿色(data[i+1])和蓝色(data[i+2])替换为0，本质上是移除它们。标准rgb三元色的红色组件保持不变，基本上使整个图像变成红色。如您所见，```applyTo```方法传递一个选项对象，该对象包含滤镜管道的该阶段的图像的数据```imageData```。从这里，我们可以遍历其像素（getImageData（）.data），以任何我们想要的方式修改它们。如果要浏览器WEBGL启用的滤镜能够在GPU上运行，要做到这一点，你必须提供一个片段着色器来描述在像素上执行的操作。在Fabric中定义了许多滤镜，您可以在其中看到如何编写片段或着色器的示例。

## 颜色（Colors）

无论您是否使用十六进制，RGB或RGBA颜色，Fabric都能提供一个完整的颜色基础，以帮助您最自然地表达自己。以下是在Fabric中定义颜色的一些方法：

```js
new fabric.Color('#f55');
new fabric.Color('#123123');
new fabric.Color('356735');
new fabric.Color('rgb(100,0,100)');
new fabric.Color('rgba(10, 20, 30, 0.5)');
```

转换也很简单。 ```toHex（）```将颜色实例转换为十六进制表示。 ```toRgb（）```可以转换为RGB，```toRgba（）```转换为带Alpha通道的RGB。

```js
new fabric.Color('#f55').toRgb(); // "rgb(255,85,85)"
new fabric.Color('rgb(100,100,100)').toHex(); // "646464"
new fabric.Color('fff').toHex(); // "FFFFFF"
```

转换不仅仅可以使单一的颜色。您也可以用另一种颜色叠加，或将其转换为灰度版本。

```js
var redish = new fabric.Color('#f55');
var greenish = new fabric.Color('#5f5');

redish.overlayWith(greenish).toHex(); // "AAAA55"
redish.toGrayscale().toHex(); // "A1A1A1"
```

## 渐变（Gradients）

更加表现出色的方式是通过渐变。渐变让我们将一种颜色混合到另一种颜色中，创造出一些令人惊叹的图形效果。

Fabric通过```setGradient```方法支持渐变，在所有对象上定义。调用```setGradient('fill', { ... })```就像设置一个对象的“fill”值一样，只不过我们用渐变而不是单色来填充对象。

```js
var circle = new fabric.Circle({
  left: 100,
  top: 100,
  radius: 50
});

circle.setGradient('fill', {
  x1: 0,
  y1: 0,
  x2: 0,
  y2: circle.height,
  colorStops: {
    0: '#000',
    1: '#fff'
  }
});
```

![ ](http://fabricjs.com/article_assets/2_15.png)

在上面的例子中，我们在100,100的位置创建一个圆圈，半径为50px。然后，我们将其填充设置为跨越整个圆形高度的渐变，从黑色到白色。

传递给一个方法的参数是一个options对象，它需要2个坐标对（x1，y1和x2，y2）以及“colorStops”对象。坐标指出了渐变开始和结束的位置，```colorStops```指出了渐变的颜色，只要它们的范围从0到1（例如0,0.1,0.3,0.5,0.75,1），您可以根据需要定义多少个```colorStops```。0表示渐变的开始，1表示结束。

坐标相对于物体左上角，因此圆的最高点为0，最低点为```circle.height```。 ```setGradient```以相同的方式计算宽度坐标（x1，x2）。

这是一个从左到右的红蓝渐变：

```js
circle.setGradient('fill', {
  x1: 0,
  y1: 0,
  x2: circle.width,
  y2: 0,
  colorStops: {
    0: "red",
    1: "blue"
  }
});
```

![ ](http://fabricjs.com/article_assets/2_16.png)

这是一个彩虹🌈渐变，颜色跨度20%：

```js
circle.setGradient('fill', {
  x1: 0,
  y1: 0,
  x2: circle.width,
  y2: 0,
  colorStops: {
    0: "red",
    0.2: "orange",
    0.4: "yellow",
    0.6: "green",
    0.8: "blue",
    1: "purple"
  }
});
```

![ ](http://fabricjs.com/article_assets/2_17.png)

你还能想到哪些炫酷的版本？

## 文本（Text）

如果你想不仅在画布上显示图像和矢量形状，还要显示文本，该怎么办？Fabric也为您准备好了！认识```fabric.Text```对象。

我们提供文本的原因有两个，首先是允许以面向对象的方式处理文本。原生canvas方法，只允许在非常低的级别上填充或描边文本。通过实例化```fabric.Text```实例，我们可以使用文本，就像我们将使用任何其他Fabric对象：移动它，缩放它，更改其属性等。

第二个原因是提供比canvas给我们更丰富的功能，包括：

- **支持多行Multiline support** 不幸的是，原生文本方法忽略了新建一行。
- **文本对齐Text alignment** 左，中，右。使用多行文本时很有用。
- **文本背景Text background** 背景也支持文本对齐。
- **文字装饰Text decoration** 下划线，上划线，贯穿线。
- **行高Line Height** 在使用多行文本时有用。
- **字符间距Char spacing** 使文本更紧凑或更间隔。
- **子范围Subranges** 将颜色和属性应用到文本对象的子对象中。
- **多字节Multibyte** 支持表情符号。
- **交互式画布编辑On canvas editing** 可以直接在画布上键入文本。

做一个hello world的例子：

```js
var text = new fabric.Text('hello world', { left: 100, top: 100 });
canvas.add(text);
```

那就对了！在画布上显示文本就像在所需位置添加```fabric.Text```实例一样简单。您可以看到，唯一必需的第一个参数是实际的文本字符串。第二个参数是通常的选项对象，它可以具有通常的left，top，填fill，opacity等属性。

当然，文本对象也有自己独有的与文本相关的属性。我们来看看其中的一些：

### 字体（fontFamily）

默认设置为“Times New Roman”，此属性允许我们更改用于呈现文本对象的字体系列。更改它将立即使文本以新字体呈现。

```js
var comicSansText = new fabric.Text("I'm in Comic Sans", {
  fontFamily: 'Comic Sans'
});
```

![ ](http://fabricjs.com/article_assets/2_5.png)

### 字体大小（fontSize）

字体大小控制渲染文本的大小。请注意，与Fabric中的其他对象不同，您不能直接更改文本的width / height属性。相反，您需要更改“fontSize”值才能使文本对象更大或更小。或者，您可以随时使用scaleX / scaleY属性。

```js
var text40 = new fabric.Text("I'm at fontSize 40", {
  fontSize: 40
});
var text20 = new fabric.Text("I'm at fontSize 20", {
  fontSize: 20
});
```

![ ](http://fabricjs.com/article_assets/2_6.png)

### 粗体（fontWeight）

```fontWeight```允许使文本更粗或更薄。就像CSS一样，您可以使用关键字（“normal”，“bold”）或数字（100,200,400,600,800）。请注意，您是否可以使用某些值取决于所选字体的粗体的可用性。如果您使用远程字体，则需要确保提供正常和粗体字体定义。

```js
var normalText = new fabric.Text("I'm a normal text", {
  fontWeight: 'normal'
});
var boldText = new fabric.Text("I'm at bold text", {
  fontWeight: 'bold'
});
```

![ ](http://fabricjs.com/article_assets/2_7.png)

### 文本修饰（textDecoration）

文本修饰允许添加下划线，上划线或贯穿线到文本。这与CSS类似，但是Fabric进一步，并允许使用上述的任何组合。所以你可以有一个文本，既有下划线又有上划线，还有贯穿线。等等。

```js
var underlineText = new fabric.Text("I'm an underlined text", {
  underline; true
});
var strokeThroughText = new fabric.Text("I'm a stroke-through text", {
  linethrough: true
});
var overlineText = new fabric.Text("I'm an overline text", {
  overline: true
});
```

![ ](http://fabricjs.com/article_assets/2_8.png)

### 阴影

此属性在版本1.3.0之前被称为“textShadow”

文本阴影由4个组件组成：颜色，水平偏移，垂直偏移和模糊大小。如果您在CSS中使用阴影，这可能看起来很熟悉。通过更改这些值可以实现许多组合。

```js
var shadowText1 = new fabric.Text("I'm a text with shadow", {
  shadow: 'rgba(0,0,0,0.3) 5px 5px 5px'
});
var shadowText2 = new fabric.Text("And another shadow", {
  shadow: 'rgba(0,0,0,0.2) 0 0 5px'
});
var shadowText3 = new fabric.Text("Lorem ipsum dolor sit", {
  shadow: 'green -5px -5px 3px'
});
```

![ ](http://fabricjs.com/article_assets/2_9.png)

### 字体风格（fontStyle）

字体样式可以是2个值之一：normal（正常）或italic（斜体）。这类似于同名的CSS属性。

```js
var italicText = new fabric.Text("A very fancy italic text", {
  fontStyle: 'italic',
  fontFamily: 'Delicious'
});
var anotherItalicText = new fabric.Text("another italic text", {
  fontStyle: 'italic',
  fontFamily: 'Hoefler Text'
});
```

![ ](http://fabricjs.com/article_assets/2_10.png)

### 描边的颜色和宽度（stroke and strokeWidth）

通过结合stroke和strokeWidth，你可以在你的文字上获得一些有趣的效果。这里有几个例子:

```js
var textWithStroke = new fabric.Text("Text with a stroke", {
  stroke: '#ff1318',
  strokeWidth: 1
});
var loremIpsumDolor = new fabric.Text("Lorem ipsum dolor", {
  fontFamily: 'Impact',
  stroke: '#c3bfbf',
  strokeWidth: 3
});
```

![ ](http://fabricjs.com/article_assets/2_11.png)


### 文本对齐（textAlign）

使用多行文本时，文本对齐很有用。使用单行文本，边界框的宽度总是与该行的宽度完全匹配，因此没有任何对齐方式。

允许的值为“left”，“center”和“right”。

```js
var text = 'this is\na multiline\ntext\naligned right!';
var alignedRightText = new fabric.Text(text, {
  textAlign: 'right'
});
```

![ ](http://fabricjs.com/article_assets/2_12.png)

### 行高（lineHeight）

CSS中可能熟悉的另一个属性是lineHeight。它允许我们改变多行文本中文本行之间的垂直间距，只是值的规格和css中的不太一样，在以下示例中，第一个文本块的lineHeight为3，第二个为1。

```js
var lineHeight3 = new fabric.Text('Lorem ipsum ...', {
  lineHeight: 3
});
var lineHeight1 = new fabric.Text('Lorem ipsum ...', {
  lineHeight: 1
});
```

![ ](http://fabricjs.com/article_assets/2_13.png)

### 文本背景颜色（textBackgroundColor）

最后，textBackgroundColor允许给文本一个背景。请注意，背景仅填充文本字符占用的空间，而不是整个边框。这意味着文本对齐改变了文本背景渲染的方式。一行的高度也是如此，因为背景是由lineHeight创建的线之间的垂直空间。

```js
var text = 'this is\na multiline\ntext\nwith\ncustom lineheight\n&background';
var textWithBackground = new fabric.Text(text, {
  textBackgroundColor: 'rgb(0,200,0)'
});
```

![ ](http://fabricjs.com/article_assets/2_14.png)

## 事件（Events）

事件驱动架构是框架内一些惊人的功能和灵活性的基础。Fabric也不例外，并提供广泛的事件系统，从低级“鼠标”事件开始到高级对象。

这些事件可以让我们在画布上发生的事件的同时，做一些事情。想知道什么时候鼠标被按下？只需要监听“mouse:down”事件，想知道什么时候对象被添加到canvas？ 监听“object:add”事件，如果想知道画布什么时候渲染完成呢？只需监听“after:render”。

事件API非常简单，类似于jQuery，Underscore.js或其他流行的JS库。有一个```on```方法来初始化事件监听器，```off```用来删除它。

让我们来看个例子：

```js
var canvas = new fabric.Canvas('...');
canvas.on('mouse:down', function(options) {
  console.log(options.e.clientX, options.e.clientY);
});
```

我们将事件“mouse:down”事件侦听器添加到画布上，并给它一个事件处理程序，用于记录事件起始位置的坐标。换句话说，它会记录鼠标按下的位置。事件处理程序接收一个选项对象，它有两个属性```e```：事件源，以及```target```：在画布上点击的对象，它有时不存在。但是这个事件在任何时候都是存在的，但是只有当你在画布上点击某个对象时，```target```才会存在。```target```也只传递给有意义的事件处理程序。例如，会传递给“mouse:down”，而不会传递给“after:render”(表示整个画布被重新绘制)。

```js
canvas.on('mouse:down', function(options) {
  if (options.target) {
    console.log('有对象被点击咯! ', options.target.type);
  }
});
```

上面的例子将记录“有对象被点击咯！”如果您单击一个对象。它还将显示点击的对象的类型。

那么Fabric中有哪些其他的事件呢？那么，从鼠标级别可以看到“mouse:down”，“mouse:move”和“mouse:up”。从通用方面看，有“after:render”。然后有选择相关的事件：“before:selection:cleared”, “selection:created”, “selection:cleared”，最后，对象的事件：“object:modified”，“object:selected”，“object:moving”，“object:scaling”，“object:rotate”，“object:added”和“object:removed”。

请注意，每当一个对象被移动（或缩放）甚至一个像素时，诸如“object:moving”（或“object:scaling”）的事件被连续地触发。另一方面，诸如“object:modified”或“selection:created”之类的事件仅在操作结束时被触发（对象修改或选择创建）。

注意我们如何将事件附加到画布上（```canvas.on（'mouse：down'，...）```）。可以想象，这意味着事件都被限定为canvas实例。如果您在页面上有多个canvas，您可以将不同的事件侦听器附加到每个canvas上。他们都是独立的，只处理分配给他们自己的事件。

为方便起见，Fabric将进一步提升事件系统，并允许您将侦听器直接附加到canvas画布中的对象上。让我们来看看：

```js
var rect = new fabric.Rect({ width: 100, height: 50, fill: 'green' });
rect.on('selected', function() {
  console.log('selected a rectangle');
});

var circle = new fabric.Circle({ radius: 75, fill: 'blue' });
circle.on('selected', function() {
  console.log('selected a circle');
});
```

我们将事件侦听器直接附加到矩形和圆形实例。使用"selected"来代替"object:selected"。同样的，使用“modified”代替“object:modified”，使用“rotating”来代替“object:rotating”。等等。。

查看此事件[演示](http://fabricjs.com/events)，以更广泛地探索Fabric的事件系统

