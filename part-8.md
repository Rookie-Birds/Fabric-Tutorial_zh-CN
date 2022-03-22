# Fabric.js介绍 第八部分

## 新的clipPath属性

在2.4.0中，我们为所有对象引入了clipPath属性。 ClipPath将替换```clipTo: funcion() {}```，试图获得相同的灵活性但更好的兼容性。

### 如何使用

创建自己的clipPath作为普通的Fabric对象，并将其指定给要剪辑的对象的clipPath属性。
根据SVG规范中的定义，clipPath没有描边，而是充满黑色，对象中与clipPath黑色像素重叠的像素将是可见的，其他像素是不可见的。

让我们从一些基本的例子开始，让我们看看它是什么样的。
在第一个示例中，一个红色的rectable被一个圆圈夹住，只有圆圈内的部分是可见的。虽然不是很有用，但是基本的功能是这样的。
请注意：clipPath位于从物体中心开始的位置，物体的originX和originY不起任何作用，而clipPath的originX和originY则起作用，定位逻辑与```fabric.Group```相同。

[查看demo](http://fabricjs.com/clippath-part1#ex1)

```js
(function() {
  var canvas = new fabric.Canvas('ex1');
  var clipPath = new fabric.Circle({
    radius: 40,
    top: -40,
    left: -40
  });
  var rect = new fabric.Rect({
    width: 200,
    height: 100,
    fill: 'red'
  });
  rect.clipPath = clipPath;
  canvas.add(rect);
})()
```

我们可以剪下一个组合：
[查看demo](http://fabricjs.com/clippath-part1#ex2)

```js
(function() {
  var canvas = new fabric.Canvas('ex2');
  var clipPath = new fabric.Circle({
    radius: 100,
    top: -100,
    left: -100
  });
  var group = new fabric.Group([
    new fabric.Rect({ width: 100, height: 100, fill: 'red' }),
    new fabric.Rect({ width: 100, height: 100, fill: 'yellow', left: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'blue', top: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'green', left: 100, top: 100 })
  ]);
  group.clipPath = clipPath;
  canvas.add(group);
})()
```

或者我们可以用组合来剪辑。在组合的情况下，记住组合中的每个对象在逻辑上都是```或```，没有```非零```或```偶数```的剪裁规则：

[查看demo](http://fabricjs.com/clippath-part1#ex3)

```js
(function() {
  var canvas = new fabric.Canvas('ex3');
  var clipPath = new fabric.Group([
    new fabric.Circle({ radius: 70, top: -70, left: -70 }),
    new fabric.Circle({ radius: 40, left: -95, top: -95 }),
    new fabric.Circle({ radius: 40, left: 15, top: 15 }),
  ], { left: -95, top: -95 });
  var group = new fabric.Group([
    new fabric.Rect({ width: 100, height: 100, fill: 'red' }),
    new fabric.Rect({ width: 100, height: 100, fill: 'yellow', left: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'blue', top: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'green', left: 100, top: 100 })
  ]);
  group.clipPath = clipPath;
  canvas.add(group);
})()
```

## 更多高级用法

### 嵌套clipPaths

clipTo的一个问题和canvas.clip()的用法是你不能有多个clipPath。
通过这种实现，clippaths可以有自己的clippaths。虽然手动编程不太直观，但它允许将clipPaths交叉到一起。

[查看demo](http://fabricjs.com/clippath-part2#ex4pre)

```js
(function() {
  var canvas = new fabric.Canvas('ex4');
  var clipPath = new fabric.Circle({ radius: 70, top: -50, left: -50 });
  var innerClipPath = new fabric.Circle({ radius: 70, top: -90, left: -90 });
  clipPath.clipPath = innerClipPath;
  var group = new fabric.Group([
    new fabric.Rect({ width: 100, height: 100, fill: 'red' }),
    new fabric.Rect({ width: 100, height: 100, fill: 'yellow', left: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'blue', top: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'green', left: 100, top: 100 })
  ]);
  group.clipPath = clipPath;
  canvas.add(group);
})()
```

组合内对象中的ClipPath应与组本身的clipPath隔离：

[查看demo](http://fabricjs.com/clippath-part2#ex5pre)

```js
(function() {
  var canvas = new fabric.Canvas('ex5');
  var clipPath = new fabric.Circle({ radius: 100, top: -100, left: -100 });
  var small = new fabric.Circle({ radius: 50, top: -50, left: -50 });
  var group = new fabric.Group([
    new fabric.Rect({ width: 100, height: 100, fill: 'red', clipPath: small }),
    new fabric.Rect({ width: 100, height: 100, fill: 'yellow', left: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'blue', top: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'green', left: 100, top: 100 })
  ]);
  group.clipPath = clipPath;
  canvas.add(group);
})()
```

### 文字剪裁

用文本进行剪裁也是不可能的，开发人员通常不得不依靠模式来实现这一点：

[查看demo](http://fabricjs.com/clippath-part2#ex6)

```js
(function() {
  var canvas = new fabric.Canvas('ex6');
  var clipPath = new fabric.Text(
    'Hi I\'m the \nnew ClipPath!\nI hope we\'ll\nbe friends',
    { top: -100, left: -100 });
  var group = new fabric.Group([
    new fabric.Rect({ width: 100, height: 100, fill: 'red' }),
    new fabric.Rect({ width: 100, height: 100, fill: 'yellow', left: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'blue', top: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'green', left: 100, top: 100 })
  ]);
  group.clipPath = clipPath;
  canvas.add(group);
})()
```

## 剪裁canvas

我们可以对静态画布应用clipPath，就像对对象一样。在这种情况下，clipPath受到缩放和平移的影响，与物体相反。clipPath被放置在左上角。

[查看demo](http://fabricjs.com/clippath-part3#ex7pre)

```js
(function() {
  var canvas = new fabric.Canvas('ex7');
  var clipPath = new fabric.Circle({ radius: 100, top: 0, left: 50 });
  var group = new fabric.Group([
    new fabric.Rect({ width: 100, height: 100, fill: 'red' }),
    new fabric.Rect({ width: 100, height: 100, fill: 'yellow', left: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'blue', top: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'green', left: 100, top: 100 })
  ]);
  canvas.clipPath = clipPath;
  canvas.add(group);
})()
```

作为旧的clipTo函数，clipPath也是剪切控件，除非你使用```canvas.controlsAboveOverlay```设置为true

[查看demo](http://fabricjs.com/clippath-part3#ex8)

```js
(function() {
  var canvas = new fabric.Canvas('ex8');
  canvas.controlsAboveOverlay = true;
  var clipPath = new fabric.Circle({ radius: 100, top: 0, left: 50 });
  var group = new fabric.Group([
    new fabric.Rect({ width: 100, height: 100, fill: 'red' }),
    new fabric.Rect({ width: 100, height: 100, fill: 'yellow', left: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'blue', top: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'green', left: 100, top: 100 })
  ]);
  canvas.clipPath = clipPath;
  canvas.add(group);
})()
```

### 动画clipPaths

剪贴簿可以像任何其他物体一样进行动画。Canvas clipPath动画非常有效，当动画对象的动画时，每次都会使对象缓存失效。

[查看demo](http://fabricjs.com/clippath-part3#ex6)

```js
(function() {
  var canvas = new fabric.Canvas('ex9');
  canvas.controlsAboveOverlay = true;
  var clipPath = new fabric.Rect({ width: 100, height: 100, top: 0, left: 0 });
  function animateLeft() {
    clipPath.animate({
      left: 200,
    }, {
      duration: 900,
      onChange: canvas.requestRenderAll.bind(canvas),
      onComplete: animateRight,
    });
  }
  function animateRight() {
    clipPath.animate({
      left: 0,
    }, {
      duration: 1200,
      onChange: canvas.requestRenderAll.bind(canvas),
      onComplete: animateLeft,
    });
  }
  function animateDown() {
    clipPath.animate({
      top: 100,
    }, {
      duration: 500,
      onChange: canvas.requestRenderAll.bind(canvas),
      onComplete: animateUp,
    });
  }
  function animateUp() {
    clipPath.animate({
      top: 0,
    }, {
      duration: 400,
      onChange: canvas.requestRenderAll.bind(canvas),
      onComplete: animateDown,
    });
  }
  var group = new fabric.Group([
    new fabric.Rect({ width: 100, height: 100, fill: 'red' }),
    new fabric.Rect({ width: 100, height: 100, fill: 'yellow', left: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'blue', top: 100 }),
    new fabric.Rect({ width: 100, height: 100, fill: 'green', left: 100, top: 100 })
  ], {
    scaleX: 1.5
  });
  animateLeft();
  animateDown();
  canvas.clipPath = clipPath;
  canvas.add(group);
})()
```
