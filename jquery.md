# jquery

## jquery core

### $ vs $()

被jquery选择器调用的方法在$.fn命名空间中使用，其接收和返回的对象都是jquery选择器的结果本身

$命名空间中的方法一般都是公用方法，一般不和选择器一起使用

### $( document ).ready()

`$()`等价于`$( document ).ready()`

`$( window ).on( "load", function() {});`会在全部内容加载完之后执行函数（包括图片，视频，音频）

### Avoiding Conflicts with Other Libraries

```js
<!-- Putting jQuery into no-conflict mode. -->
<script src="prototype.js"></script>
<script src="jquery.js"></script>
<script>
 
var $j = jQuery.noConflict();
// $j is now an alias to the jQuery function; creating the new alias is optional.
 
$j(document).ready(function() {
    $j( "div" ).hide();
});
 
// The $ variable now has the prototype meaning, which is a shortcut for
// document.getElementById(). mainDiv below is a DOM element, not a jQuery object.
window.onload = function() {
    var mainDiv = $( "main" );
}
 
</script>
```

### 不重要

- .attr()
- 选择器 :eq() :gt() :lt()
- 使用选择器 链式调用 .eq(2).end()
- 操作元素 getOrSet attr val html text width 移动、复制、删除元素
- Traversing（遍历）
- css,styling & Dimensions toggleClass() height() innerHeight(pad) outerHeight(pad+border+mar)
- Data Methods $( "#myDiv" ).data( "keyName", { foo: "bar" } );

### jquery Object

封装dom元素后对其提供新的api,跟原生的方法比增强了兼容性和方便程度。获取jquery对象的方法有两种，一种是$()，一种是jquery选择器，每一个jquery对象都是唯一的，===时为false，及时这两个对象由同一个选择器生成出来，但是取出的两个原生dom元素是===的，jquery对象不会随着元素的变化变化，除非重新声明一个新的对象

.eq()从jquery对象集合中中获取一个jquery对象，get() []从jquery对象集合中获取的是dom元素，

### Utility Methods

- $.trim()

```js
// Returns "lots of extra whitespace"
$.trim( "    lots of extra whitespace    " );
```

- $.each()

```js
$.each([ "foo", "bar", "baz" ], function( idx, val ) {
    console.log( "element " + idx + " is " + val );
});
 
$.each({ foo: "bar", baz: "bim" }, function( k, v ) {
    console.log( k + " : " + v );
});
```

- $.inArray()

```js
var myArray = [ 1, 2, 3, 5 ];
 
if ( $.inArray( 4, myArray ) !== -1 ) {
    console.log( "found it!" );
}
```

- $.extend()

```js
var firstObject = { foo: "bar", a: "b" };
var secondObject = { foo: "baz" };
 
var newObject = $.extend( firstObject, secondObject );
 
console.log( firstObject.foo ); // "baz"
console.log( newObject.foo ); // "baz"
```

- $.proxy()

```js
var myFunction = function() {
    console.log( this );
};
var myObject = {
    foo: "bar"
};
 
myFunction(); // window
 
var myProxyFunction = $.proxy( myFunction, myObject );
 
myProxyFunction(); // myObject
```

- type

```js
$.isArray([]); // true
$.isFunction(function() {}); // true
$.isNumeric(3.14); // true

$.type( true ); // "boolean"
$.type( 3 ); // "number"
$.type( "test" ); // "string"
$.type( function() {} ); // "function"
 
$.type( new Boolean() ); // "boolean"
$.type( new Number(3) ); // "number"
$.type( new String('test') ); // "string"
$.type( new Function() ); // "function"
 
$.type( [] ); // "array"
$.type( null ); // "null"
$.type( /test/ ); // "regexp"
$.type( new Date() ); // "date"
```

### 迭代

- $.each()

```js
$.each( arr, function( index, value ){
    sum += value;
});

console.log( sum ); // 15
```

- .each()

```js
$( "li" ).each( function( index, element ){
    console.log( $( this ).text() );
});
```

- `$.each()`只迭代简单对象，`.each()`迭代的是jquery对象
- 某些时候`.each()`不是必须的，比如使用`addClass()`时可以不用`.each()`，需要用的方法有这些：`.attr() (getter)，.css() (getter)，.data() (getter)，.height() (getter)，.html() (getter)，.innerHeight()，.innerWidth()，.offset() (getter)，.outerHeight()，.outerWidth()，.position()，.prop() (getter)，.scrollLeft() (getter)，.scrollTop() (getter)，.val() (getter)，.width() (getter)`，set时入参为匿名函数且其入参为index和value并返回值的，等价于`.each()`
- `$.map() & .map()`

## Events

### jQuery Event Basics

- `.click()`是`.on()`的简写

- 事件不会更新，当节点更新时，需要重新绑定

- event object 包含了如下的内容pageX, pageY, type, which, data, target, namespace等等

- 绑定多个事件

```js
// Multiple events, same handler
$( "input" ).on(
    "click change", // Bind handlers for multiple events
    function() {
        console.log( "An input was clicked or changed!" );
    }
);

// Binding multiple events with different handlers
$( "p" ).on({
    "click": function() { console.log( "clicked!" ); },
    "mouseover": function() { console.log( "hovered!" ); }
});
```

- 解除事件绑定

```js
$( "p" ).off( "click" );
// Tearing down a particular click handler, using a reference to the function
var foo = function() { console.log( "foo" ); };
var bar = function() { console.log( "bar" ); };
 
$( "p" ).on( "click", foo ).on( "click", bar );
$( "p" ).off( "click", bar ); // foo is still bound to the click event
```

- 绑定的方法只执行一次

```js
/ Switching handlers using the `.one()` method
$( "p" ).one( "click", firstClick );
 
function firstClick() {
    console.log( "You just clicked this for the first time!" );
 
    // Now set up the new handler for subsequent clicks;
    // omit this step if no further click responses are needed
    $( this ).click( function() { console.log( "You have clicked this before!" ); } );
}
```

## 源码阅读