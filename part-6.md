# Fabric.js介绍 第六部分

了解转换如何在fabricJS上工作是尽可能顺利地编写应用程序的关键方面。

## 与转换相关的方法和属性

如果您计划理解和使用与自定义代码一起使用的fabricJS转换，那么这些方法应该是您最应该学习使用的方法。
一般来说，在本页面中，我们将矩阵称为6个数字的数组，表示平面上的转换，并将其作为一个类似{x: number, y: number}简单的JS对象或者是fabric.Point类的实例。(通常没有区别)

```
Canvas:
- vieportTransform = matrix;
Objects:
- matrix = fabric.Object.prototype.calcTransformMatrix();
- matrix = fabric.Object.prototype.calcOwnMatrix();
Utils:
- point = fabric.util.transformPoint(point, matrix);
- matrix = fabric.util.multiplyTransformMatrices(matrix, matrix);
- matrix = fabric.util.invertTransform(matrix);
- options = fabric.util.qrDecompose(matrix);
```

## 从一个空间移动到另一个空间（空间变换）
使用fabricJS常常需要与坐标和位置交互，但如果没有正确的背景，理解这些坐标的位置可能会很麻烦。
我将列出变换及其用法，然后我将尝试制作两个例子来更好地阐明发生了什么，以及如何做到这一点。

Canvas.viewportTransform，将虚拟画布的一点移动到缩放和平移空间。
在cooridantes可以找到一个点，当画布不缩放和不平移时，在viewportTransfrom M中应用缩放和平移后，在位置P处的位置可以找到：

```js
newP = fabric.util.transformPoint(P, canvas.viewportTransform);
```

Object.calcTransformMatrix，将表示特定时刻的通用对象转换(受顶部、左侧、缩放和许多其他属性的影响)的矩阵还原，并将点从对象空间移动到画布空间，而不是缩放。因此，给定物体空间坐标中的一个点在坐标P处，这个点将在画布上的坐标为：

```js
newP = fabric.util.transformPoint(P, object.calcTransformMatrix());
```

## 变形顺序
渲染期间的Fabric按此顺序应用转换：

```
zoom and pan => object transformation => nested object ( group ) => nested object deeper ( nested groups )
```

## 恢复顺序
invertTransform实用程序用于在转换逻辑中移回以进行一些反向计算：
假设您想要在画布上标记一个对象，单击鼠标，单击点。点击点P，例如元素上的10,10像素。对象被缩放和旋转，画布被缩放和平移。
要反转渲染计算，您可以遵循以下逻辑：

```js
// 计算应用于对象像素的总转换：
var mCanvas = canvas.viewportTransform;
var mObject = object.calcTransformMatrix();
var mTotal = fabric.util.multiplyTransformMatrices(mCanvas, mObject); // 反转顺序会产生错误的结果
var mInverse = fabric.util.invertTransform(mTotal);
var pointInObjectPixels = fabric.util.transformPoint(pointClicked, mInverse);
```

现在，```pointInObjectPixels```是一个位于坐标空间中的点，其中```0,0```位于对象的中心。

## 了解矩阵的效果

给定top，left，angle，scaleX，scaleY，skewX，skewY，flipX，flipY相对简单，可以创建表示该转换的矩阵。 不直接的是如何回去。矩阵有6个维度，有6个数字，而属性是7，因为我们可以按比例缩放。确实存在无数个矩阵，但是可能的属性组合的数量是一个无限大的。
现在开始使用```fabric.util.qrDecompose(matrix)```可以为我们解码矩阵。给定函数的通用可逆矩阵，它返回包含这些信息的选项对象：

```js
{
  angle: number, // 度
  scaleX: number,
  scaleY: number,
  skewX: number, // 度
  skewY: 0, // 总是0度
  translateX: number,
  translateY: number,
}
```

这个函数给出了这个矩阵的一个可能解，将skewY约束为0。

## 一个真实的用例

一个开发人员想要将对象分组，但同时让它们自由。理想情况下，当主要对象移动时，他希望其他一些对象跟随它。
为了解释这个例子，我将调用主要对象BOSS和其他MINIONS

假设画布周围有一些物体我们可以自由移动它们。在某一点上，我们要锁定它们的相对位置和比例尺，并移动一个。当我们设置我们想要的位置时，BOSS位置由矩阵描述，正如我们到目前为止所学到的，以及每个MINIONS。

我确信它存在一个矩阵，它定义了从BOSS移动到MINIONS的必要转换，我必须找到它。

```js
// 我在寻找未知关系矩阵，其中:
BOSS * UNKNOW = MINION
// 我向左乘以BOSS-INV
BOSS-INV * BOSS * UNKNOW = BOSS-INV * MINION
// BOSS-INV * BOSS = IDENTIY, 一个中立的矩阵。
IDENTITY * UNKNOW = BOSS-INV * MINION
// so...
UNKNOW = BOSS-INV * MINION
// 在fabricJS代码中等于:
var minions = canvas.getObjects().filter(o => o !== boss);
var bossTransform = boss.calcTransformMatrix();
var invertedBossTransform = fabric.util.invertTransform(bossTransform);
minions.forEach(o => {
  var desiredTransform = multiply(invertedBossTransform, o.calcTransformMatrix());
  // 在这里保存所需的关系。
  o.relationship = desiredTransform;
});
```

好的，现在我知道如何找到关系，我可以编写一些事件处理程序来在每个BOSS操作上应用这种关系。
[查看demo](http://fabricjs.com/using-transformations#bind)

```js
var canvas = new fabric.Canvas('c');
var boss = new fabric.Rect(
  { width: 150, height: 200, fill: 'red' });
var minion1 = new fabric.Rect(
  { width: 40, height: 40, fill: 'blue' });
var minion2 = new fabric.Rect(
  { width: 40, height: 40, fill: 'blue' });

canvas.add(boss, minion1, minion2);

boss.on('moving', updateMinions);
boss.on('rotating', updateMinions);
boss.on('scaling', updateMinions);

var multiply = fabric.util.multiplyTransformMatrices;
var invert = fabric.util.invertTransform;

function updateMinions() {
  var minions = canvas.getObjects().filter(o => o !== boss);
  minions.forEach(o => {
    if (!o.relationship) {
      return;
    }
    var relationship = o.relationship;
    var newTransform = multiply(
      boss.calcTransformMatrix(),
      relationship
    );
    opt = fabric.util.qrDecompose(newTransform);
    o.set({
      flipX: false,
      flipY: false,
    });
    o.setPositionByOrigin(
      { x: opt.translateX, y: opt.translateY },
      'center',
      'center'
    );
    o.set(opt);
    o.setCoords();
  });
}

document.getElementById('bind').onclick = function() {
  var minions = canvas.getObjects().filter(o => o !== boss);
  var bossTransform = boss.calcTransformMatrix();
  var invertedBossTransform = invert(bossTransform);
  minions.forEach(o => {
    var desiredTransform = multiply(
      invertedBossTransform,
      o.calcTransformMatrix()
    );
    o.relationship = desiredTransform;
  });
}
    
```

