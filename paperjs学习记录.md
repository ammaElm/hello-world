# Paperjs学习记录

## Paperjs中的几个基本概念

在`Paper.js`中, 每个项目或者上下文都需要一系列的对象去描述它的状态，例如当前活动的项目、活动视图及有效工具等。这涉及到三个概念，`program（项目`）、`view（视图）`、`tool（工具）`。

+ program

+ view

  像 `onFrame()`、`onResize()` 这样全局的处理函数（都属于`view`的类），需要手动挂载在处理对象上。

  ```js
  paper.install(window);
  window.onload = function() { 
  	paper.setup('myCanvas'); 
  	let path = new Path.Rectangle([75, 75], [100, 100]); 
  	path.strokeColor = 'black'; 
    view.onframe = function(){
      path.rotate(3);
    }
  }
  ```

+ tool

  在`javaScript`中，需要自己创建工具：

  ```js
  paper.install(window);
  window.onload = function() {
    paper.setup('myCanvas'); 
    //创建一个简单的绘图工具：
    let tool = new Tool(); 
    let path;
  
    //定义mousedown和mousedrag处理程序
    tool.onMouseDown = function(event) { 
      path = new Path(); 
      path.strokeColor = 'black';
      path.add(event.point); 
    } 
  
    tool.onMouseDrag = function(event) {
      path.add(event.point); 
    }
  }
  ```

  

## 几何

`paperjs`中，`Point`，`Size`，`Rectangle`是描述几何图形属性的基本类。

+ Point

  `Point`对象描述啦二维位置，它有两个属性`x`和`y`，即坐标。

  ```js
  new Point (x,y)
  ```

  

+ Size

  `Size`对象描述二维空间中的抽象维度，它有两个属性`width`和`height`，即宽度和高度。

  ```js
  new Size(width,height)
  ```

  

+ Rectangle

  `Rectangle`对象可以描述为`Point`对象和`Size`对象的组合，描述二维位置和大小。因此它具有`x` ， `y` ， `宽度`和`高度`这四个属性。

  ```js
  new Rectangle(x,y,width,height)
  ```

  Rectangle对象比Point和Size对象稍微复杂一些，并暴露了一系列额外的中心和角点对象： `center` ， `topLeft` ， `topRight` ， `bottomLeft` ， `bottomRight` ，`leftCenter` ， `topCenter` ， `rightCenter`和`bottomCenter` 。

  ```js
  var rect = new Rectangle(); 
  console.log(rect);
  // {x：0，y：0，width：0，height：0}
  //现在我们可以举例说明它的大小......
  rect.size = new Size(100, 200); 
  // 及其中心
  rect.center = new Point(100, 100);
  ```

  

  作为定义矩形尺寸的替代数字方式，Rectangle对象还公开以下属性，与`x` ， `y`， `width`和`height相对` ： `left` ， `top` ， `right`和`bottom` ：

  ```js
  var rect = new Rectangle(); 
  rect.left = 100; 
  rect.right = 200; 
  rect.bottom = 400;
  rect.top = 200; 
  console.log(rect);
  ```

  ### 数学运算

  `Paper.js`中`point`的点可以直接进行加、减、乘、除运算。

  ```js
  var point1 = new Point(10, 20);
  var point2 = new Point(10, 20)*4;
  var point4 = point3 + 30;
  ```

  `Paper.js`提供了几种用于舍入点和大小值的数学函数：

  ```js
  var point = new Point(1.2, 1.8);
  var rounded = point.round(); 
  // { x: 1, y: 2 }
  
  var ceiled = point.ceil（）;
  // { x: 2, y: 2 }
  
  var floored = point.floor（）;
  // { x: 1, y: 1 }
  
  //使用Point.random（）或Size.random（）为0到1之间的每个属性创建一个具有随机值的点或大小：
  var point = new Point(50, 100) * Point.random();
  ```

  ### 矢量几何

  `Vector几何`是`Paper.js`中的一等公民。

  在许多方面，`Paper.js`中的向量与点非常相似。 两者都由`x`和`y`坐标表示。 但是其中还是有一定的区别：点描述绝对位置，而矢量代表相对信息; 即从一个点到另一个点的方法。 

  ```JS
  var point1 = new Point(50, 50); 
  var point2 = new Point(110, 200);
  var vector = point2 - point1;
  ```

  `Paper.js`中的所有角度都以度为单位进行测量，并且顺时针方向。 

## 路径

`Paperjs`使用`新的Path（）`构造函数创建一个新的路径项，并使用`path.add（segment）`函数向其添加段。

```js
var myPath = new Path();
myPath.strokeColor = 'black';
myPath.add(new Point(0, 0));
myPath.add(new Point(100, 50));

//要插入与现有段相关的段，可以使用path.insert（index，segment）函数
myPath.insert(1, new Point(30, 40));
```

>#### 请注意：
>
>`Point`对象表示Paper.js项目的二维空间中的一个点。 它不是指路径中的锚点。 将`Point`传递给`add`或`insert`等函数时，它会动态转换为`Segment` 。

Paper.js允许使用`path.smooth（）`函数自动平滑路径。

关闭路径，可以将其`path.closed`属性设置为`true` 。

要从路径中删除段，使用`path.removeSegment（index）`函数并将其传递给要删除的段的索引。

```js
var myCircle = new Path.Circle(new Point(100, 70), 50);
myCircle.strokeColor = 'black';
myCircle.selected = true;

myCircle.removeSegment(0);
```

<img src="/Users/amaris_elm/Library/Application Support/typora-user-images/image-20190509171048841.png" style="width:200px">

<img src="/Users/amaris_elm/Library/Application Support/typora-user-images/image-20190509171115374.png" style="width:200px">

要从项目中完全删除项目，我们使用`item.remove（）`

### 正多边形路径

要创建规则的多边形路径，我们使用`新的Path.RegularPolygon（center，sides，radius）`构造函数。

`center`参数描述多边形的中心点， `sides`参数描述多边形具有的边数量，`radius`参数描述多边形的半径。

```js
var triangle = new Path.RegularPolygon(new Point(80, 70), 3, 50);
triangle.fillColor = '#e9e9ff';
triangle.selected = true;

// Create a decagon shaped path 
var decagon = new Path.RegularPolygon(new Point(200, 70), 10, 50);
decagon.fillColor = '#e9e9ff';
decagon.selected = true;
```

<img src="/Users/amaris_elm/Library/Application Support/typora-user-images/image-20190509172543621.png" style="width:300px">

### 平滑，简化和展平

paper.js提供了两种不同的平滑路径的方法。

+ `path.smooth（）`通过更改其段句柄而不添加或删除段点来平滑路径。

+ `path.simplify（）`通过分析`path.segments`数组并用更优化的段集替换它来平滑路径，从而减少内存使用并加快绘图速度。

`path.flatten（error）`将路径中的曲线转换为具有最大指定错误的直线。 保证结果行不会超出`error`参数指定的数量。

```js
// Create a circle shaped path at { x: 80, y: 50 }
// with a radius of 35:
var path = new Path.Circle({
	center: [80, 50],
	radius: 35
});

// Select the path, so we can inspect its segments:
path.selected = true;

// Create a copy of the path and move it by 150 points:
var copy = path.clone();
copy.position.x += 150;

// Flatten the copied path, with a maximum error of 4 points:
copy.flatten(4);
```

<img src="/Users/amaris_elm/Library/Application Support/typora-user-images/image-20190509174340988.png" style="width:300px;">

## 交互工具

### 鼠标工具

Paper.js提供了不同鼠标行为的处理函数。

```JS
function onMouseDown(event) {
	console.log('You pressed the mouse!');
}

function onMouseDrag(event) {
	console.log('You dragged the mouse!');
}

function onMouseUp(event) {
	console.log('You released the mouse!');
}
```

鼠标处理函数接受包含一系列鼠标事件的时间对象，例如鼠标的当前位置（ `event.point` ），按下鼠标的位置（ `event.downPoint` ）等。

```js
//绘制一条曲线
var myPath;

function onMouseDown(event) {
	myPath = new Path();
	myPath.strokeColor = 'black';
}

function onMouseDrag(event) {
	myPath.add(event.point);
}

function onMouseUp(event) {
	var myCircle = new Path.Circle({
		center: event.point,
		radius: 10
	});
	myCircle.strokeColor = 'black';
	myCircle.fillColor = 'white';
}
```

### 键盘工具

`Paper.js`通过定义`onKeyDown`或`onKeyUp`处理函数来接收相关按键信息。

```js
// Create a centered text item at the center of the view:
var text = new PointText({
	point: view.center,
	content: 'Click here to focus and then press some keys.',
	justification: 'center',
	fontSize: 15
});

function onKeyDown(event) {
	// When a key is pressed, set the content of the text item:
	text.content = 'The ' + event.key + ' key was pressed!';
}

function onKeyUp(event) {
	// When a key is released, set the content of the text item:
	text.content = 'The ' + event.key + ' key was released!';
}
```



## 项目使用

在一个`Paper.js`的项目中可以出现以下几大类，图层、路径、复合路径、组、文本项、栅格等。

每种项目都有特定的行为：路径包含点，层可以是活动的，组和复合路径有子。

但由于它们都是物品，因此它们也有许多共同的行为。 所有这些共享行为都可以在`Item`参考中找到。

创建新项目时，它们会自动添加到`project.activeLayer`的`item.children`列表的末尾。

### 更改项目

#### 更改项目位置

可以通过更改其`item.position`属性来移动项目中的项目。 这会使项目的中心点移动。

要使项目跟随鼠标，我们可以简单地将其位置设置为鼠标的位置：

```js
var circlePath = new Path.Circle(new Point(50, 50), 25);
circlePath.fillColor = 'black'

function onMouseMove(event) {
	circlePath.position = event.point;
}
```

如果想知道项目的宽度和高度，我们可以查询`item.bounds`属性的`宽度`和`高度` ：

```js
var circlePath = new Path.Circle(new Point(50, 50), 25); 
console.log(circlePath.bounds.width);

console.log(circlePath.bounds.topLeft);
// { x: 25.0, y: 25.0 } 
console.log(circlePath.bounds.topRight);
// { x: 75.0, y: 25.0 } 
console.log(circlePath.bounds.bottomRight);
// { x: 75.0, y: 75.0 }
console.log(circlePath.bounds.bottomLeft);
// { x: 25.0, y: 75.0 } 
```

<img src="/Users/amaris_elm/Library/Application Support/typora-user-images/image-20190509183551412.png" style="width:300px">



<img src="/Users/amaris_elm/Library/Application Support/typora-user-images/image-20190509183649306.png" style="width:300px;">

 #### 缩放项目

要按项目的相同数量缩放项目的宽度和高度，可以调用项目的`item.scale（scale）` 。

#### 旋转物品

要旋转一个项目，我们调用`item.rotate（角度）`函数，并将它以我们想要旋转的角度传递给它。 这将按顺时针方向旋转项目角度。

```js
var path = new Path.Rectangle(new Point(50, 50), new Size(100, 50));
path.style = {
	fillColor: 'white',
	strokeColor: 'black'
};

// Create a copy of the path and set its stroke color to red:
var copy = path.clone();
copy.strokeColor = 'red';

// Rotate the copy by 45 degrees:
copy.rotate(45);
```

### 项目层次结构

`Paper.js`项目的结构基于堆叠顺序原则。

`Paper.js`具有图层列表：`project.layers` 。 每当重绘视图时，`Paper.js`都会遍历这些图层中的项目并按照外观顺序绘制它们。

+ `activeLayer`活动层

  每个Paper.js `项目`都以一个`Layer`开始，可以通过`project.activeLayer`访问。所有新创建的项目将自动添加为当前活动的图层的子项。

+ `children`子项

  我们将其他项目中包含的项目称为其子项。 可以通过其`item.children`数组访问图层中包含的子项 。

#### 按名称访问每一项

如果某个项目有名称，则可以通过其父项的子项列表按名称访问该项目：

```js
var path = new Path.Circle(new Point(80, 50), 35);
// Set the name of the path:
path.name = 'example';

// Save a reference to the children array in a variable,
// so we don't end up with very long lines of code:
var children = project.activeLayer.children;

// The path can be accessed by name:
children['example'].fillColor = 'red';
```



#### 图层和组

可以将`图层`和`组`项视为硬盘中的文件夹。 它们将项目组合在一起，对它们执行的任何操作都会直接更改其中包含的项目。

`图层`和`组`几乎相同。 主要区别在于可以激活图层项目，这意味着创建的任何新项目都会自动放置在其中。 无论何时创建新`图层` ，它都会成为`项目的project.activeLayer`。

创建组时，它还没有任何子组。 您可以通过几种不同的方式将子项添加到组中：

```js
// Create two circle shaped paths:
var path = new Path.Circle(new Point(80, 50), 35);
var secondPath = new Path.Circle(new Point(120, 50), 35);

var group = new Group([path, secondPath]);

// Change the fill color of the items contained within the group:
group.style = {
	fillColor: 'red',
	strokeColor: 'black'
};
```

还可以使用`item.addChild（item）`函数在创建组后添加子组：

```js
// Create two circle shaped paths:
var path = new Path.Circle(new Point(80, 50), 35);
var secondPath = new Path.Circle(new Point(180, 50), 35);

// Create an empty group:
var group = new Group();

// Add the paths to the group:
group.addChild(path);
group.addChild(secondPath);

// Change the fill color of the items contained within the group:
group.fillColor = 'green';
```

要将子项插入到特定索引处的组或层中，可以使用`item.insertChild（index，item）`函数：

```js
var redPath = new Path.Circle(new Point(80, 50), 30);
redPath.fillColor = 'red';

var greenPath = new Path.Circle(new Point(100, 50), 30);
greenPath.fillColor = 'green';

// Insert the green path at index 0 of the children
// array of the active layer:
project.activeLayer.insertChild(0, greenPath);

```

要从`Paper.js`文档中删除项目，可以调用其`item.remove（）`函数。

如果项目具有子项（即，它是具有子项的图层，组或其他类型的项目），则其所有子项也将从项目中删除。

要删除项目中包含的所有子项，可以调用`item.removeChildren（）`。

## 动画

可以使用`onFrame`处理程序在Paper.js中创建动画。

```js
var path = new Path.Rectangle({
	point: [75, 75],
	size: [75, 75],
	strokeColor: 'black'
});

function onFrame(event) {
	// Each frame, rotate the path by 3 degrees:
	path.rotate(3);
}
```

