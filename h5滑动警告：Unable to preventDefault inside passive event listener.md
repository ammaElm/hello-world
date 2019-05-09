# 解决滚动警告：Unable to preventDefault inside passive event listener

## 起因

近期更新项目，使用`fullpage.js`实现单页滚动，页面报错：

![16a9633d7ccbb200](/Users/amaris_elm/demo/notice/media/16a9633d7ccbb200.jpg)

## passive event listener

关于 `passive event listener` 的问题已遇到过多次。

当我们滚动页面的时候或者需要监听 `touch` 事件的时候，浏览器无法预先知道一个事件处理函数中会不会调用 `preventDefault()`，故而通过一个延迟，大约300ms的停顿来检测，整个页面就显得卡顿。

为了优化这一点，自chrome51开始，`passive event listener`被引入。

通过对 `addEventListener `第三个参数设置 `{ passive : true }` 来避免浏览器检查是否在`touch`事件`handler`中调用了`preventDefault`。

这种情形下，如果我们依旧调用`preventDefault`，就会在控制台打印一个警告，告诉我们`passive event listener` 中的` preventDefault`会被忽略。

通常解决的方法有两个：

+ `addEventlistener `第三个参数传入`{ passive : false }`

  ```js
  document.addEventListener(
    'touchstart',
    function(event){
      event.preventDafault();
    },
    { passive: false }
  );
  ```

  

+ 触发全局的`css`样式`touch-action`，例如

  ```js
  * { touch-action: pan-y; } 
  //表示手指头可以垂直移动
  ```



## touch-action

由于是第三方的插件，选择了全局css样式的覆盖。

>在 Chrome（版本 55 及更高版本）、Internet Explorer 和 Edge 中，`PointerEvents` 是建议的自定义手势实现方法。
>
>`PointerEvents` 的一大特色是，它将包括鼠标、触摸和触控笔事件在内的多种输入类型合并成一个回调集。需要侦听的事件是 `pointerdown`、`pointermove`、`pointerup` 和 `pointercancel`。
>
>其他浏览器中的对应项是 `touchstart`、`touchmove`、`touchend` 和 `touchcancel` 触摸事件，如果想为鼠标输入实现相同的手势，则需实现 `mousedown`、`mousemove` 和 `mouseup`。
>
>使用这些事件需要对 DOM 元素调用 `addEventListener()` 方法，使用的参数为事件名称、回调函数和一个布尔值。布尔值决定是否应在其他元素有机会捕获并解释事件之前或之后捕获事件。（`true` 表示想要先于其他元素捕获事件）
>
>```js
>// Check if pointer events are supported.
>if (window.PointerEvent) {
>  // Add Pointer Event Listener
>  swipeFrontElement.addEventListener('pointerdown', this.handleGestureStart, true);
>  swipeFrontElement.addEventListener('pointermove', this.handleGestureMove, true);
>  swipeFrontElement.addEventListener('pointerup', this.handleGestureEnd, true);
>  swipeFrontElement.addEventListener('pointercancel', this.handleGestureEnd, true);
>} else {
>  // Add Touch Listener
>  swipeFrontElement.addEventListener('touchstart', this.handleGestureStart, true);
>  swipeFrontElement.addEventListener('touchmove', this.handleGestureMove, true);
>  swipeFrontElement.addEventListener('touchend', this.handleGestureEnd, true);
>  swipeFrontElement.addEventListener('touchcancel', this.handleGestureEnd, true);
>
>  // Add Mouse Listener
>  swipeFrontElement.addEventListener('mousedown', this.handleGestureStart, true);
>}
>```



`touch-action` 支持的关键字有：

```js
touch-action: auto;
touch-action: none;
touch-action: pan-x;
touch-action: pan-left;
touch-action: pan-right;
touch-action: pan-y;
touch-action: pan-up;
touch-action: pan-down;
touch-action: pinch-zoom;
touch-action: manipulation;
```

## 结束语

`Unable to preventDefault inside passive event listener`这个问题出现有一段时间，没想到可以通过touch-action相关css属性去解决，邂逅touch-action相关小知识，撒一波花~



