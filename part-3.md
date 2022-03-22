# Fabric.js介绍 第三部分

我们已经介绍了本系列的第一部分和第二部分的大部分基础知识。我们继续前进到更高级的技巧！

## 组合（Group）

![ ](http://fabricjs.com/article_assets/8.png)

我们首先谈论的是组合。组合是Fabric最强大的功能之一。 将任何Fabric对象分组成一个单一实体的简单方法，为什么要这样做？当然是为了能够将这些对象作为一个单元来处理。

可以用鼠标来将画布上的任何数量的Fabric对象进行分组，形成单一的组合形式，一旦分组，组内Fabric对象可以一起移动，甚至一起修改。他们组成一个组。我们可以缩放这个组合，旋转，甚至改变其表现性质：颜色、透明度、边界等。

这正是组合所要做的，每当您在画布上看到这样的选择时，Fabric将在内部创建一组对象。只有以编程方式提供使用组的访问才有意义。这就是是fabric的组合。

让我们创建一个包含两个Fabric对象的组合，圆和文本：

```js
var circle = new fabric.Circle({
  radius: 100,
  fill: '#eef',
  scaleY: 0.5,
  originX: 'center',
  originY: 'center'
});

var text = new fabric.Text('hello world', {
  fontSize: 30,
  originX: 'center',
  originY: 'center'
});

var group = new fabric.Group([ circle, text ], {
  left: 150,
  top: 100,
  angle: -10
});

canvas.add(group);
```

首先，我们创建了一个“hello world”文本对象。将```originX```和```originY```设置为“center”会使得将以组合为中心，默认情况下，组合成员相对于组合的左上角定位。然后，圆圈以100px半径填充“#eef”颜色并垂直缩放0.5倍（scaleY = 0.5）。然后，我们创建了一个```fabric.Group```实例，通过一个包含这两个Fabric对象的数组传递给这个组合，并给它的位置为150/100和-10角度。最后，将这个组合添加到画布（使用```canvas.add()```）。

您会看到画布上的一个对象，看起来像一个椭圆标签。请注意，如果我们要修改他们的样子，可以将该他们作为单个实体使用，我们只是改变组合的属性，给出要更改的的```left```，```top```和```angle```值。

![ ](http://fabricjs.com/article_assets/3_1.png)

让我们再来修改一下canvas画布上的这个组合：

```js
group.item(0).set('fill', 'red');
group.item(1).set({
text: 'trololo',
fill: 'white'
});
```

这里发生了什么？我们通过```item()```方法访问组中的各个对象，并修改其属性。第一个对象是椭圆，第二个是文字。让我们看看发生了什么：

![ ](http://fabricjs.com/article_assets/3_2.png)

您现在可能注意到的一件重要的事情是，组合中的对象都相对于组合的中心定位，当我们改变了text对象中的内容时，即使内容的宽度改变了，但是它仍然保持居中的位置，如果你不想要这个行为，您需要指定对象的left/right坐标。在这种情况下，根据这些坐标将它们分组在一起。

让我们创建并组合3个圆，使它们相互水平定位：

```js
var circle1 = new fabric.Circle({
  radius: 50,
  fill: 'red',
  left: 0
});
var circle2 = new fabric.Circle({
  radius: 50,
  fill: 'green',
  left: 100
});
var circle3 = new fabric.Circle({
  radius: 50,
  fill: 'blue',
  left: 200
});

var group = new fabric.Group([ circle1, circle2, circle3 ], {
  left: 200,
  top: 100
});

canvas.add(group);
```

![ ](http://fabricjs.com/article_assets/3_3.png)

使用团体时要注意的另一件事是**对象的状态**，举个例子，当使用图片形成组合时，您需要确保这些图像已经完全加载。由于Fabric已经提供了帮助方法来确保图像被加载，这变得相当容易：

```js
fabric.Image.fromURL('/assets/pug.jpg', function(img) {
  var img1 = img.scale(0.1).set({ left: 100, top: 100 });

  fabric.Image.fromURL('/assets/pug.jpg', function(img) {
    var img2 = img.scale(0.1).set({ left: 175, top: 175 });

    fabric.Image.fromURL('/assets/pug.jpg', function(img) {
      var img3 = img.scale(0.1).set({ left: 250, top: 250 });

      canvas.add(new fabric.Group([ img1, img2, img3], { left: 200, top: 200 }))
    });
  });
});
```

![ ](http://fabricjs.com/article_assets/3_4.png)

在使用组合时可以使用哪些其他方法?使用```getObjects()```方法，可以返回一个包含组合中所有对象的数组，使用```size()```可以知道组合中所有对象的数量。使用```contains()```可以检查某个对象是否在组合中。我们之前看到的```item()```可以检索组中的特定对象。使用```forEachObject```可以遍历每个组合中的对象。还有```add()```和```remove()```方法来相应地从组中添加和删除对象。

您可以通过2种方式添加/删除组中的对象：更新组合的尺寸/位置，我们建议使用更新尺寸，除非您正在执行批量处理操作，并且在进程中组合的宽度/高度都没有问题。

在组合中心添加一个长方形：

```js
group.add(new fabric.Rect({
  ...
  originX: 'center',
  originY: 'center'
}));
```

在组合的中心附近100px添加矩形：

```js
group.add(new fabric.Rect({
  ...
  left: 100,
  top: 100,
  originX: 'center',
  originY: 'center'
}));
```

在组合中心添加矩形并且更新组合的尺寸：

```js
group.addWithUpdate(new fabric.Rect({
  ...
  left: group.get('left'),
  top: group.get('top'),
  originX: 'center',
  originY: 'center'
}));
```

在组合的中心附近100px添加矩形并且更新组合的尺寸：

```js
group.addWithUpdate(new fabric.Rect({
  ...
  left: group.get('left') + 100,
  top: group.get('top') + 100,
  originX: 'center',
  originY: 'center'
}));
```

最后，如果您想创建一个已经存在于画布上的对象的组合，则需要首先克隆它们：

```js
// 创建一个包含两个已存在对象的副本的组合
var group = new fabric.Group([
  canvas.item(0).clone(),
  canvas.item(1).clone()
]);

// 移除所有对象并且重新渲染
canvas.clear().renderAll();

// 将组合添加到canvas画布
canvas.add(group);
```

## 序列化

一旦你开始构建一种有状态的应用程序，也许允许用户在服务器上保存画布内容的结果，或者将内容流传输到不同的客户端，你都需要**canvas序列化**。如果仍然要发送画布内容，也是可以的，有一个选项可以将画布导出到图像。但是上传图片到服务器无疑是相当占用带宽的，论大小，没有什么可以比得过文本了，这就是为什么Fabric为canvas画布序列化/反序列化提供了极好的支持。

### toObject, toJSON

Fabric中的canvas序列化方法主要是```toObject()```和```toJSON()```方法。我们来看一个简单的例子，首先序列化一个空的画布：

```js
var canvas = new fabric.Canvas('c');
JSON.stringify(canvas);
// '{"objects":[],"background":"rgba(0, 0, 0, 0)"}'
```

我们使用ES5 ```JSON.stringify()```方法，如果```toJSON```方法存在，则会隐性调用。由于Fabric中的canvas实例已经具有```toJSON```方法，就相当于我们调用了```canvas.toJSON()```。

注意返回的表示空的canvas画布的字符串。它是JSON形式，并且由“objects”和“background”属性组成。“objects”当前为空，因为canvas上没有任何内容，而background的默认值为“rgba（0，0，0，0）”）。

让我们给画布不同的背景，看看它有什么变化：

```js
canvas.backgroundColor = 'red';
JSON.stringify(canvas);
// '{"objects":[],"background":"red"}'
```

正如所料，画布表现现在反映出新的背景颜色。现在，我们来添加一些Fabric对象！

```js
canvas.add(new fabric.Rect({
  left: 50,
  top: 50,
  height: 20,
  width: 20,
  fill: 'green'
}));
console.log(JSON.stringify(canvas));
```

输出是：

```
'{"objects":[{"type":"rect","left":50,"top":50,"width":20,"height":20,"fill":"green","overlayFill":null,"stroke":null,"strokeWidth":1,"strokeDashArray":null,"scaleX":1,"scaleY":1,"angle":0,"flipX":false,"flipY":false,"opacity":1,"selectable":true,"hasControls":true,"hasBorders":true,"hasRotatingPoint":false,"transparentCorners":true,"perPixelTargetFind":false,"rx":0,"ry":0}],"background":"rgba(0, 0, 0, 0)"}'
```

看起来改变了好多，在“objects”数组新增了一个对象，序列化为JSON。请注意它的表示方式是如何包含其所有视觉特征：left, top, width, height, fill, stroke等。

如果我们要添加另一个对象，比如，一个位于矩形旁边的红色圆圈，您会看到相应改变：

```js
canvas.add(new fabric.Circle({
  left: 100,
  top: 100,
  radius: 50,
  fill: 'red'
}));
console.log(JSON.stringify(canvas));
```

输出是：

```
'{"objects":[{"type":"rect","left":50,"top":50,"width":20,"height":20,"fill":"green","overlayFill":null,"stroke":null,"strokeWidth":1,"strokeDashArray":null,"scaleX":1,"scaleY":1,"angle":0,"flipX":false,"flipY":false,"opacity":1,"selectable":true,"hasControls":true,"hasBorders":true,"hasRotatingPoint":false,"transparentCorners":true,"perPixelTargetFind":false,"rx":0,"ry":0},{"type":"circle","left":100,"top":100,"width":100,"height":100,"fill":"red","overlayFill":null,"stroke":null,"strokeWidth":1,"strokeDashArray":null,"scaleX":1,"scaleY":1,"angle":0,"flipX":false,"flipY":false,"opacity":1,"selectable":true,"hasControls":true,"hasBorders":true,"hasRotatingPoint":false,"transparentCorners":true,"perPixelTargetFind":false,"radius":50}],"background":"rgba(0, 0, 0, 0)"}'
```

注意看```“type”：“rect”```和```“type”：“circle”```部分，即使起初看起来可能会有很多输出，但是与图像序列化所得到的结果无关。只是为了比较，让我们来看看你将用```canvas.toDataURL({format: 'png'})``(译者注：这个命令原版有误)`获得的一个字符串的大约1/10（！）

```
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAyAAAAK8CAYAAAAXo9vkAAAgAElEQVR4Xu3dP4xtBbnG4WPAQOQ2YBCLK1qpoQE1/m+NVlCDwUACicRCEuysrOwkwcJgAglEItRQaWz9HxEaolSKtxCJ0FwMRIj32zqFcjm8e868s2fNWo/Jygl+e397rWetk5xf5pyZd13wPwIECBAgQIAAAQIECBxI4F0H+hwfQ4AAAQIECBAgQIAAgQsCxENAgAABAgQIECBAgMDBBATIwah9EAECBAgQIECAAAECAsQzQIAAAQIECBAgQIDAwQQEyMGofRABAgQIECBAgAABAgLEM0CAAAECBAgQIECAwMEEBMjBqH0QAQIECBAgQIAAAQICxDNAgAABAgQIECBAgMDBBATIwah9EAECBAgQIECAAAECAsQzQIAAAQIECBAgQIDAwQQEyMGofRABAgQIECBAgAABAgLEM0CAAAECBAgQIECAwMEEBMjBqH0QAQIECBAgQIAAAQICxDNAgAABAgQIECBAgMDBBATIwah9EAECBAgQIECAAAECAsQzQIAAAQIECBAgQIDAwQQEyMGofRABAgQIECBAgAABAgLEM0CAAAECBAgQIECAwMEEBMjBqH0QAQIECBAgQIAAAQICxDNAgAABAgQIECBAgMDBBATIwah9EAECBAgQIECAAAECAsQzQIAAAQIECBAgQIDAwQQEyMGofRABAgQIECBAgAABAgLEM0CAAAECBAgQIECAwMEEBMjBqH0QAQIECBAgQIAAAQICxDNAgAABAgQIECBAgMDBBATIwah9EAECBAgQIECAAAECAsQzQIAAAQIECBAgQIDAwQQEyMGofRABAgQIECBAgAABAgLEM0CAAAECBAgQIECAwMEEBMjBqH0QAQIECBAgQIAAAQICxDNAgAABAgQIECBAgMDBBATIwah9EAECBAgQIECAAAECAsQzQIAAAQIECBAgQIDAwQQEyMGofRABAgQIECBAgAABAgLEM0CAAAECBAgQIECAwMEEBMjBqH0QAQIECBAgQIAAAQICxDNAgAABAgQIECBAgMDBBATIwah9EAECBAgQIECAAAECyw+Qb134R/U2fevC8q+5esGWESBAgAABAgQIEFiOwPL/MC5AlvO0OBMCBAgQIECAAAECJxQQICcE9HYCBAgQIECAAAECBPYXECD7W3klAQIECBAgQIAAAQInFBAgJwT0dgIECBAgQIAAAQIE9hcQIPtbeSUBAgQIECBAgAABAicUECAnBPR2AgQIECBAgAABAgT2FxAg+1t5JQECBAgQIECAAAECJxQQICcE9HYCBAgQIECAAAECBPYXECD7W3klAQIECBAgQIAAAQInFBAgJwTc9+3z49yvmNd+dI7PzPHJOW6Y4wNzXD3HlXNc9pZdb85/vzbHK3P8aY7n5vj1HL+Y43dz417f97O9jgABAgQIECBAgMBSBATIKd2JCY5dWNwyx5fn+PwcV5U/6tXZ99M5fjjHk3Mjd6HifwQIECBAgAABAgQWLSBAirdnouP6WXfvHHfOcU1x9T6rXp4XPTLHA3NTX9jnDV5DgAABAgQIECBA4NACAuSE4hMdl8+Kr83xzTmuO+G61ttfnEXfnuN7c4PfaC21hwABAgQIECBAgMBJBQTIJQpOeFw7b71/jtsvccWh3vbYfNB9c6NfOtQH+hwCBAgQIECAAAECFxMQIMd8No7C4+F5283HfOtZv/ypOYG7hMhZ3wafT4AAAQIECBDYtoAA2fP+H/1Vqwd3f4jf8y1Lfdkunu7xV7OWenucFwECBAgQIEBg3QICZI/7O/Fxx7xs9wf3t36r3D3evciX7L7F7+6rIY8u8uycFAECBAgQIE
```

你可能会想知道为什么还有```toObject```。很简单，```toObject```返回与```toJSON```相同的表示形式，只能以实际对象的形式，而不用字符串序列化。例如，以一个绿色矩形的canvas的之前的例子。

```canvas.toObject()```的输出是这样的：

```json
{ "background" : "rgba(0, 0, 0, 0)",
  "objects" : [
    {
      "angle" : 0,
      "fill" : "green",
      "flipX" : false,
      "flipY" : false,
      "hasBorders" : true,
      "hasControls" : true,
      "hasRotatingPoint" : false,
      "height" : 20,
      "left" : 50,
      "opacity" : 1,
      "overlayFill" : null,
      "perPixelTargetFind" : false,
      "scaleX" : 1,
      "scaleY" : 1,
      "selectable" : true,
      "stroke" : null,
      "strokeDashArray" : null,
      "strokeWidth" : 1,
      "top" : 50,
      "transparentCorners" : true,
      "type" : "rect",
      "width" : 20
    }
  ]
}
```

可以看出，```toJSON```输出本质上是一个字符串的```toObject```输出。现在，有趣的（有用的）事情是，toObject输出是聪明和懒惰的。您在“objects”数组中看到的内容是遍历canvas画布上所有的对象并调用该对象本身自己的```toObject```方法的结果。```fabric.Path```有自己的```toObject```，返回路径的“points”数组，
```fabric.Image```也有自己的```toObject```，返回图像的“src”属性。以真正的面向对象的方式，所有对象都能够自己序列化。

这意味着当您创建自己的“类”，或者需要定制对象的序列化表示时，您需要做的就是使用```toObject```方法 - 完全替换它或扩展它。我们来试试一下：

```js
var rect = new fabric.Rect();
rect.toObject = function() {
  return { name: 'trololo' };
};
canvas.add(rect);
console.log(JSON.stringify(canvas));
```

输出：

```
'{"objects":[{"name":"trololo"}],"background":"rgba(0, 0, 0, 0)"}'
```

您可以看到，objects数组现在有一个自定义的展示。这种覆盖可能不是很有用 - 尽管把重点提到了：我们如何拓展和更改矩形的```toObject```方法与额外的属性。

```js
var rect = new fabric.Rect();

rect.toObject = (function(toObject) {
  return function() {
    return fabric.util.object.extend(toObject.call(this), {
      name: this.name
    });
  };
})(rect.toObject);

canvas.add(rect);

rect.name = 'trololo';

console.log(JSON.stringify(canvas));
```

输出：

```
'{"objects":[{"type":"rect","left":0,"top":0,"width":0,"height":0,"fill":"rgb(0,0,0)","overlayFill":null,"stroke":null,"strokeWidth":1,"strokeDashArray":null,"scaleX":1,"scaleY":1,"angle":0,"flipX":false,"flipY":false,"opacity":1,"selectable":true,"hasControls":true,"hasBorders":true,"hasRotatingPoint":false,"transparentCorners":true,"perPixelTargetFind":false,"rx":0,"ry":0,"name":"trololo"}],"background":"rgba(0, 0, 0, 0)"}'
```

我们使用附加属性“name”来扩展对象的现有```toObject```方法，所以“name”属性是```toObject```输出的一部分，值得一提的还有，如果你扩展这样的对象，您还需要确保对象所属的“class”（在这种情况下为```fabric.Rect```）在“stateProperties”数组中具有此属性，以便从字符串表示中加载canvas将解析并将其正确添加到对象中。

您可以将对象标记为不可导出设置```excludeFromExport```为```true```。以这种方式，一些在canvas上用来辅助的对象在序列化过程中不会被保存。

### toSVG

另一种高效的基于文本的画布表示是SVG格式。由于Fabric专门从事画布上的SVG解析和渲染，所以将其作为双向过程并提供画布到SVG转换是有意义的。让我们将相同的矩形添加到画布中，并查看从toSVG方法返回的表示形式：

```js
canvas.add(new fabric.Rect({
  left: 50,
  top: 50,
  height: 20,
  width: 20,
  fill: 'green'
}));
console.log(canvas.toSVG());
```

输出：

```
'<?xml version="1.0" standalone="no" ?><!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 20010904//EN" "http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd"><svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="800" height="700" xml:space="preserve"><desc>Created with Fabric.js 0.9.21</desc><rect x="-10" y="-10" rx="0" ry="0" width="20" height="20" style="stroke: none; stroke-width: 1; stroke-dasharray: ; fill: green; opacity: 1;" transform="translate(50 50)" /></svg>'
```

就像用```toJSON```和```toObject```一样，```toSVG```在canvas上调用时，将其逻辑委托给每个单独的对象，并且每个单独的对象都有自己的toSVG方法，它是特殊的对象类型。如果您需要修改或扩展对象的SVG表示，那么可以使用```toSVG```做同样的事情，就像我们对```toObject```一样。

与Fabric的专有的```toObject / toJSON```相比，SVG表示的好处是可以将其投放到任何支持SVG的渲染器（浏览器，应用程序，打印机，相机等）中，并且它应该可以正常工作。但是，您首先需要将其加载到canvas画布上。谈到在画布上加载东西，现在我们可以将画布序列化成一个有效的文本块，我们将如何加载到画布上？

## 反序列化，SVG解析器

与序列化类似，有两种方式从字符串加载canvas：从JSON表示，或从SVG。当使用JSON表示时，使用```loadFromJSON```和```oadFromDatalessJSON```方法。SVG时，使用```fabric.loadSVGFromURL```和```fabric.loadSVGFromString```。

请注意，前2个方法是实例方法，直接在canvas实例上调用，而最后2个方法是静态方法，在“fabric”对象上而不是在canvas上调用。

这些方法没有什么可说的。他们的工作正如你所期望的那样。例如，我们先从画布中获取以前的JSON输出并将其加载到干净的画布上：

```js
var canvas = new fabric.Canvas();

canvas.loadFromJSON('{"objects":[{"type":"rect","left":50,"top":50,"width":20,"height":20,"fill":"green","overlayFill":null,"stroke":null,"strokeWidth":1,"strokeDashArray":null,"scaleX":1,"scaleY":1,"angle":0,"flipX":false,"flipY":false,"opacity":1,"selectable":true,"hasControls":true,"hasBorders":true,"hasRotatingPoint":false,"transparentCorners":true,"perPixelTargetFind":false,"rx":0,"ry":0},{"type":"circle","left":100,"top":100,"width":100,"height":100,"fill":"red","overlayFill":null,"stroke":null,"strokeWidth":1,"strokeDashArray":null,"scaleX":1,"scaleY":1,"angle":0,"flipX":false,"flipY":false,"opacity":1,"selectable":true,"hasControls":true,"hasBorders":true,"hasRotatingPoint":false,"transparentCorners":true,"perPixelTargetFind":false,"radius":50}],"background":"rgba(0, 0, 0, 0)"}',() => { });
```

两个物体“神奇”出现在画布上：

![ ](http://fabricjs.com/article_assets/3_5.png)

所以从字符串加载画布是很容易的。但是那个奇怪的```loadFromDatalessJSON```方法是干什么的？与我们刚刚使用的```loadFromJSON```有什么不同？为了理解为什么我们需要这种方法，我们需要看一下具有或多或少的复杂路径对象的序列化画布。像这个：

![ ](http://fabricjs.com/article_assets/3_6.png)

而这个形状的```JSON.stringify(canvas)```输出是：

```
{"objects":[{"type":"path","left":184,"top":177,"width":175,"height":151,"fill":"#231F20","overlayFill":null,"stroke":null,"strokeWidth":1,"strokeDashArray":null,"scaleX":1,"scaleY":1,"angle":-19,"flipX":false,"flipY":false,"opacity":1,"selectable":true,"hasControls":true,"hasBorders":true,"hasRotatingPoint":false,"transparentCorners":true,"perPixelTargetFind":false,"path":[["M",39.502,61.823],["c",-1.235,-0.902,-3.038,-3.605,-3.038,-3.605],["s",0.702,0.4,3.907,1.203],["c",3.205,0.8,7.444,-0.668,10.114,-1.97],["c",2.671,-1.302,7.11,-1.436,9.448,-1.336],["c",2.336,0.101,4.707,0.602,4.373,2.036],["c",-0.334,1.437,-5.742,3.94,-5.742,3.94],["s",0.4,0.334,1.236,0.334],["c",0.833,0,6.075,-1.403,6.542,-4.173],["s",-1.802,-8.377,-3.272,-9.013],["c",-1.468,-0.633,-4.172,0,-4.172,0],["c",4.039,1.438,4.941,6.176,4.941,6.176],["c",-2.604,-1.504,-9.279,-1.234,-12.619,0.501],["c",-3.337,1.736,-8.379,2.67,-10.083,2.503],["c",-1.701,-0.167,-3.571,-1.036,-3.571,-1.036],["c",1.837,0.034,3.239,-2.669,3.239,-2.669],["s",-2.068,2.269,-5.542,0.434],["c",-3.47,-1.837,-1.704,-8.18,-1.704,-8.18],["s",-2.937,5.909,-1,9.816],["C",34.496,60.688,39.502,61.823,39.502,61.823],["z"],["M",77.002,40.772],["c",0,0,-1.78,-5.03,-2.804,-8.546],["l",-1.557,8.411],["l",1.646,1.602],["c",0,0,0,-0.622,-0.668,-1.691],["C",72.952,39.48,76.513,40.371,77.002,40.772],["z"],["M",102.989,86.943],["M",102.396,86.424],["c",0.25,0.22,0.447,0.391,0.594,0.519],["C",102.796,86.774,102.571,86.578,102.396,86.424],["z"],["M",169.407,119.374],["c",-0.09,-5.429,-3.917,-3.914,-3.917,-2.402],["c",0,0,-11.396,1.603,-13.086,-6.677],["c",0,0,3.56,-5.43,1.69,-12.461],["c",-0.575,-2.163,-1.691,-5.337,-3.637,-8.605],["c",11.104,2.121,21.701,-5.08,19.038,-15.519],["c",-3.34,-13.087,-19.63,-9.481,-24.437,-9.349],["c",-4.809,0.135,-13.486,-2.002,-8.011,-11.618],["c",5.473,-9.613,18.024,-5.874,18.024,-5.874],["c",-2.136,0.668,-4.674,4.807,-4.674,4.807],["c",9.748,-6.811,22.301,4.541,22.301,4.541],["c",-3.097,-13.678,-23.153,-14.636,-30.041,-12.635],["c",-4.286,-0.377,-5.241,-3.391,-3.073,-6.637],["c",2.314,-3.473,10.503,-13.976,10.503,-13.976],["s",-2.048,2.046,-6.231,4.005],["c",-4.184,1.96,-6.321,-2.227,-4.362,-6.854],["c",1.96,-4.627,8.191,-16.559,8.191,-16.559],["c",-1.96,3.207,-24.571,31.247,-21.723,26.707],["c",2.85,-4.541,5.253,-11.93,5.253,-11.93],["c",-2.849,6.943,-22.434,25.283,-30.713,34.274],["s",-5.786,19.583,-4.005,21.987],["c",0.43,0.58,0.601,0.972,0.62,1.232],["c",-4.868,-3.052,-3.884,-13.936,-0.264,-19.66],["c",3.829,-6.053,18.427,-20.207,18.427,-20.207],["v",-1.336],["c",0,0,0.444,-1.513,-0.089,-0.444],["c",-0.535,1.068,-3.65,1.245,-3.384,-0.889],["c",0.268,-2.137,-0.356,-8.549,-0.356,-8.549],["s",-1.157,5.789,-2.758,5.61],["c",-1.603,-0.179,-2.493,-2.672,-2.405,-5.432],["c",0.089,-2.758,-1.157,-9.702,-1.157,-9.702],["c",-0.8,11.75,-8.277,8.011,-8.277,3.74],["c",0,-4.274,-4.541,-12.82,-4.541,-12.82],["s",2.403,14.421,-1.336,14.421],["c",-3.737,0,-6.944,-5.074,-9.879,-9.882],["C",78.161,5.874,68.279,0,68.279,0],["c",13.428,16.088,17.656,32.111,18.397,44.512],["c",-1.793,0.422,-2.908,2.224,-2.908,2.224],["c",0.356,-2.847,-0.624,-7.745,-1.245,-9.882],["c",-0.624,-2.137,-1.159,-9.168,-1.159,-9.168],["c",0,2.67,-0.979,5.253,-2.048,9.079],["c",-1.068,3.828,-0.801,6.054,-0.801,6.054],["c",-1.068,-2.227,-4.271,-2.137,-4.271,-2.137],["c",1.336,1.783,0.177,2.493,0.177,2.493],["s",0,0,-1.424,-1.601],["c",-1.424,-1.603,-3.473,-0.981,-3.384,0.265],["c",0.089,1.247,0,1.959,-2.849,1.959],["c",-2.846,0,-5.874,-3.47,-9.078,-3.116],["c",-3.206,0.356,-5.521,2.137,-5.698,6.678],["c",-0.179,4.541,1.869,5.251,1.869,5.251],["c",-0.801,-0.443,-0.891,-1.067,-0.891,-3.473],...
```

...这只是整个输出的其中一部分！

这里发生了什么？事实证明，这个```fabric.Path```实例（这个形状）包括数百条贝塞尔线，决定了它是如何被渲染的。JSON表示中的所有这些```[“c”，0,2.67，-0.979,5.253，-2.048,9.079]```数据片段对应于这些曲线中的每一个。而当数百（甚至数千）的这些数据片段构成画布的表现最终是相当巨大的。

该怎么办？ 
这时使用```toDatalessJSON```会方便很多。我们来试试吧：

```js
canvas.item(0).sourcePath = '/assets/dragon.svg';
console.log(JSON.stringify(canvas.toDatalessJSON()));
```

输出：

```
{"objects":[{"type":"path","left":143,"top":143,"width":175,"height":151,"fill":"#231F20","overlayFill":null,"stroke":null,"strokeWidth":1,"strokeDashArray":null,"scaleX":1,"scaleY":1,"angle":-19,"flipX":false,"flipY":false,"opacity":1,"selectable":true,"hasControls":true,"hasBorders":true,"hasRotatingPoint":false,"transparentCorners":true,"perPixelTargetFind":false,"path":"/assets/dragon.svg"}],"background":"rgba(0, 0, 0, 0)"}
```

这样看起来小多了，发生了什么？注意在调用```toDatalessJSON```之前做了什么，我们给了“sourcePath”属性一个路径：“/assets/dragon.svg”，然后，当我们调用```toDatalessJSON```时，从前一个输出（这几百个路径命令）的整个堆积如山的路径字符串被一个单独的“dragon.svg”字符串所取代。你可以看到它显示在上面。

当使用大量复杂的形状时，```toDessessJSON```可以让我们进一步减少画布表现，并用简单的SVG链接代替巨大的路径数据表示。

现在回到```loadFromDatalessJSON```方法...你可以猜到，它只是允许从无数据版本的canvas表示中加载canvas。```loadFromDatalessJSON``几乎非常了解如何使用这些“路径”字符串（如“/assets/dragon.svg”），加载它们，并用作相应路径对象的数据。

现在，我们来看看SVG加载方法。我们可以使用字符串或URL：

```js
fabric.loadSVGFromString('...', function(objects, options) {
  var obj = fabric.util.groupSVGElements(objects, options);
  canvas.add(obj).renderAll();
});
```

第一个参数是SVG字符串，第二个是回调函数，当SVG被解析和加载时调用回调，并且接收到2个参数 - ```objects```和```option```，```objects```包含从SVG路径解析的对象数组，路径组（复杂对象），图像，文本等。为了将所有这些对象分组成一个连贯的集合，并使它们与SVG文档中的方式相同，我们使用```fabric.util.groupSVGElements```传递```objects```和```option```。在返回值中，我们可以获得```fabric.Path```或```fabric.Group```的一个实例，然后我们可以将其添加到画布上。

```fabric.loadSVGFromURL```的工作方式相同，除了将SVG内容的字符串替换为URL，还要注意，Fabric将尝试通过XMLHttpRequest获取该URL，因此SVG需要符合通常的SOP（标准操作程序）规则。

## 子类

由于Fabric以真正的面向对象的方式构建，它旨在使子类化和扩展简单自然。从这个系列的第1部分你知道，Fabric中存在对象的现有层次结构。所有2D对象（路径，图像，文本等），还有一些“classes”，都继承自```fabric.Object```，像```fabric.IText```，甚至形成3级继承。

那么我们怎么去把Fabric中的一个现有的“class”分类呢？或者甚至创造自己的？

对于这个任务，我们需要使用```fabric.util.createClass```实用程序。```createClass```只不过是对JavaScript的原型继承的简单抽象。我们先创建一个简单的Point“class”：

```js
var Point = fabric.util.createClass({
  initialize: function(x, y) {
    this.x = x || 0;
    this.y = y || 0;
  },
  toString: function() {
    return this.x + '/' + this.y;
  }
});
```

```createClass```接受一个对象并使用该对象的属性创建具有实例级属性的“class”。唯一需要特别处理的属性是“initialize”用作构造函数（相当于constructor）。所以现在，当初始化Point时，我们将创建一个带有“x”和“y”属性的实例，和“toString”方法：

```js
var point = new Point(10, 20);

point.x; // 10
point.y; // 20

point.toString(); // "10/20"

```

如果我们想创建一个“Point”类的子类“ColoredPoint”，我们将使用createClass：

```js
var ColoredPoint = fabric.util.createClass(Point, {
  initialize: function(x, y, color) {
    this.callSuper('initialize', x, y);
    this.color = color || '#000';
  },
  toString: function() {
    return this.callSuper('toString') + ' (color: ' + this.color + ')';
  }
});
```

注意如何将具有实例级属性的对象作为第二个参数传递，而第一个参数接收```Point```“class”，这告诉```createClass```将其用作这个父类的“class”。为了避免重复，我们使用的是```callSuper```方法，它调用父类“class”的方法。这意味着如果我们要改变```Point```，这些更改也会传播到```ColoredPoint```。看```ColoredPoint```的行为：

```js
var redPoint = new ColoredPoint(15, 33, '#f55');

redPoint.x; // 15
redPoint.y; // 33
redPoint.color; // "#f55"

redPoint.toString(); "15/35 (color: #f55)"
```

所以现在我们去创建我们自己的“类”和“子类”，我们来看看如何使用已经存在的Fabric。例如，我们创建一个LabeledRect“类”，它基本上是一个与它相关联的一些标签的矩形。当在画布上渲染时，该标签将被表示为矩形内的文本。类似于以前的组合示例与圆和文本。当您使用Fabric时，您会注意到，可以通过使用组合或使用自定义类来实现这样的组合抽象。

```js
var LabeledRect = fabric.util.createClass(fabric.Rect, {

  type: 'labeledRect',

  initialize: function(options) {
    options || (options = { });

    this.callSuper('initialize', options);
    this.set('label', options.label || '');
  },

  toObject: function() {
    return fabric.util.object.extend(this.callSuper('toObject'), {
      label: this.get('label')
    });
  },

  _render: function(ctx) {
    this.callSuper('_render', ctx);

    ctx.font = '20px Helvetica';
    ctx.fillStyle = '#333';
    ctx.fillText(this.label, -this.width/2, -this.height/2 + 20);
  }
});
```

在这里似乎还有很多事情，但实际上很简单。

首先，我们将父类“class”指定为```fabric.Rect```，以利用其渲染能力。接下来，我们定义“type”属性，将其设置为“labeledRect”。这只是为了一致，因为所有的Fabric对象都有类型属性（rect，circle，path，text等）。然后，我们已经熟悉的构造函数（```initialize```），我们再次使用```callSuper```。此外，我们将对象的标签设置为通过选项传递的值。最后，我们剩下两个方法：```toObject```和```_render```。 ```toObject```，正如您从序列化章节已经知道的那样，负责实例的对象（和JSON）表示。由于LabeledRect具有与常规```Rect```相同的属性，也是一个标签，因此我们将扩展父类的```toObject```方法，并在其中添加标签。最后但并非最不重要的是，```_rendermethod```是实际绘制实例的原因。还有一个```callSuper```调用，它是渲染矩形，有3行文本渲染逻辑。

现在，如果我们要渲染这样的对象：

```js
var labeledRect = new LabeledRect({
  width: 100,
  height: 50,
  left: 100,
  top: 100,
  label: 'test',
  fill: '#faa'
});
canvas.add(labeledRect);
```

我们会得到这个：

![ ](http://fabricjs.com/article_assets/3_7.png)

更改标签值或任何其他的矩形属性将按预期工作：

```js
labeledRect.set({
  label: 'trololo',
  fill: '#aaf',
  rx: 10,
  ry: 10
});
```

![ ](http://fabricjs.com/article_assets/3_8.png)

当然，在这一点上，你可以随意修改这个“类”的行为。例如，将某些值设置为默认值，以避免每次传递给构造函数。或者使实例上的某些可配置属性可用。如果您使其他属性可配置，则可能需要在```toObject```和```initialize```中对其进行计算。

```js
...
initialize: function(options) {
  options || (options = { });

  this.callSuper('initialize', options);

  // 给所有标注的矩形固定宽/高100/50
  this.set({ width: 100, height: 50 });

  this.set('label', options.label || '');
}
...
_render: function(ctx) {

  // 使标签的字体和填充值可配置

  ctx.font = this.labelFont;
  ctx.fillStyle = this.labelFill;

  ctx.fillText(this.label, -this.width/2, -this.height/2 + 20);
}
...
```

在这个说明中，我正在包装这个系列的第三部分，其中我们分为了Fabric的一些更先进的方面。在群组，课程和(反)序列化的帮助下，您可以将您的应用程序提升到一个全新的水平。