title: 深入理解JavaScript事件系统
tags: [javascript,js-event]
categories: javascript
---

# 深入理解JavaScript事件系统
之前研究重写alert/confirm时，曾遇到一问题：js如何绑定和销毁匿名方法事件。怀着这个问题，一起来研究一下js的事件系统。

## DOM0事件模型
DOM0事件模型就是直接在Dom对象上注册事件名称，所有浏览器均支持，形如
```javascript
document.getElementById("btn").onclick = function (e) {
    alert('test');
}
// 一个意思
document.getElementById("btn")["onclick"] = function (e) {
    alert('test');
}
// 当然通过函数式调用也是一样
var btn = document.getElementById("btn");
btn.onclick = function (e) {
    alert('test');
}
// 解除事件
btn.onclick = null;

// 但如果多次绑定同一事件，后绑定的将覆盖先绑定的，Demo中有验证

```

## DOM2事件模型
DOM2事件模型将事件通过`addEventListener`和`removeEventListener`管理，新增了事件冒泡和捕获，同时也支持元素绑定多个事件，要注意的是低版本IE必须用`attachEvent`代替。jquery 1.10以前是这样实现事件的绑定：

```javascript
// Bind the global event handler to the element
// 在元素上绑定全局事件处理
if ( elem.addEventListener ) {
    elem.addEventListener( type, eventHandle, false );

} else if ( elem.attachEvent ) {
    // IE9以下兼容(IE确实有点自娱自乐的感觉)
    elem.attachEvent( "on" + type, eventHandle );
}
```

解除绑定也做了兼容处理：

```javascript
jQuery.removeEvent = document.removeEventListener ?
    function( elem, type, handle ) {
        if ( elem.removeEventListener ) {
            // 第三个参数false是作用于冒泡阶段(addEventListener相同)
            // 第三个参数true是作用于事件捕获阶段
            elem.removeEventListener( type, handle, false );
        }
    } :
    function( elem, type, handle ) {
        var name = "on" + type;
        // 如果浏览器支持detachEvent
        if ( elem.detachEvent ) {

            // #8545, #7054, preventing memory leaks for custom events in IE6-8
            // detachEvent needed property on element, by name of that event, to properly expose it to GC
            // IE6-8自定义事件存在内存泄露问题，解除事件时需要释放引用，深入研究内存泄露(http://www.cnblogs.com/fsjohnhuang/p/4455822.html)
            if ( typeof elem[ name ] === core_strundefined ) {
                elem[ name ] = null;
            }
            // 解除事件
            elem.detachEvent( name, handle );
        }
    };
```

## 事件系统
既然了解事件绑定模型，书写一个简要的事件处理系统也不是一个难事。
```javascript
// 引用自冰极峰<http://www.cnblogs.com/binyong/articles/1750263.html>
var EventUtil = {
  //注册
  addHandler: function(element, type, handler){
    if (element.addEventListener){
      element.addEventListener(type, handler, false);
    } else if (element.attachEvent){
      element.attachEvent("on" + type, handler);
    } else {
      element["on" + type] = handler;
    }
  },
  //移除注册
  removeHandler: function(element, type, handler){
    if (element.removeEventListener){
        element.removeEventListener(type, handler, false);
    } else if (element.detachEvent){
        element.detachEvent("on" + type, handler);
    } else {
        element["on" + type] = null;
    }
  }
};
```

上面代码不难理解，一般js类库(插件)也是这样实现。说到这，不得不提[Dean Edwards](http://dean.edwards.name/weblog/2005/10/add-event/)大神,来看看他是怎么实现的:

```javascript
function addEvent(element, type, handler) {
  // assign each event handler a unique ID
  // 给每个事件赋予一个唯一ID
  if (!handler.$$guid) handler.$$guid = addEvent.guid++;
  // create a hash table of event types for the element
  // 创建一个事件的hash表
  if (!element.events) element.events = {};
  // create a hash table of event handlers for each element/event pair
  // 为每个元素/事件建立一个hash表
  var handlers = element.events[type];
  if (!handlers) {
    handlers = element.events[type] = {};
    // store the existing event handler (if there is one)
    // 如果事件列表中不存在该类型的事件，挂载至事件列表
    if (element["on" + type]) {
      handlers[0] = element["on" + type];
    }
  }
  // store the event handler in the hash table
  // 保存事件句柄到hash表(这里很巧妙，$$guid是个唯一自增的索引)
  handlers[handler.$$guid] = handler;
  // assign a global event handler to do all the work
  // 赋值一个全局事件处理
  element["on" + type] = handleEvent;
};
// a counter used to create unique IDs
// 初始化计数器
addEvent.guid = 1;

function removeEvent(element, type, handler) {
  // delete the event handler from the hash table
  if (element.events && element.events[type]) {
    // 注意这里是delete，不是赋null
    delete element.events[type][handler.$$guid];
  }
};

// 事件处理方法
function handleEvent(event) {
  // grab the event object (IE uses a global event object)
  // 获取事件对象(IE下获取全局事件对象)
  event = event || window.event;
  // get a reference to the hash table of event handlers
  // 获取当前类型的事件队列
  var handlers = this.events[event.type];
  // execute each event handler
  //  执行每个处理函数
  for (var i in handlers) {
    this.$$handleEvent = handlers[i];
    this.$$handleEvent(event);
  }
};
```

不知道大家注意到没有，Dean Edwards并没有使用`addEventListener`来绑定事件，触发事件也是自己实现的`handleEvent`方法，为何这样？

回到最开始探究的问题（js如何绑定匿名方法事件），返回到`EventUtil`，看`移除注册`这里。注意`removeHandler`第三个参数，必须有`handler`。那么问题来了，如果绑定的时候`EventUtil.addHandler(doc, 'click', function () {})`，如何移除`click`事件？答案自然有了。

## 题外话
上面处理事件的方法，并不是完全体，最新请看[这里](http://dean.edwards.name/my/events.js):
```javascript
// 调用的地方改为
event = event || fixEvent(((this.ownerDocument || this.document || this).parentWindow || window).event);

function fixEvent(event) {
  // add W3C standard event methods
  // 添加标准的W3C事件方法(主要针对IE)
  // 阻止浏览器默认行为
  event.preventDefault = fixEvent.preventDefault;
  // 阻止冒泡
  event.stopPropagation = fixEvent.stopPropagation;
  return event;
};
fixEvent.preventDefault = function() {
  this.returnValue = false;
};
fixEvent.stopPropagation = function() {
  this.cancelBubble = true;
};
```
为啥增加这个补丁（为啥要阻止这两个事件？试试给submit绑定click事件就明白了）？且看IE/FF两种事件的区别：
```javascript
// IE
window.event.cancelBubble = true;//停止冒泡
window.event.returnValue = false;//阻止事件的默认行为

// FF
event.preventDefault();//阻止事件的默认行为
event.stopPropagation();//阻止事件的传播
```
巧妙的解决IE下兼容问题。