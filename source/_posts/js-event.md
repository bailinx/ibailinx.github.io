# 深入理解JavaScript事件系统
之前研究重写alert/confirm时，曾遇到一问题：js如何绑定和销毁匿名事件。怀着这个问题，一起来研究一下js的事件系统。

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

```

## DOM2事件模型
DOM2事件模型将事件通过'addEventListener'和'removeEventListener'管理。jquery 1.9.X以前是这样实现绑定：

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
var E = {
    //添加事件
    add : function(el, type, fn){
        if(el.addEventListener){
            el.addEventListener(type, fn, false);
        }else{
            el['e'+fn] = function(){
                fn.call(el,evt);
            };
            el.attachEvent('on' + type, el['e'+fn]);
        }
    },
    //删除事件
    remove : function(el, type, fn){
        if(el.removeEventListener){
            el.removeEventListener(type, fn, false);
        }else if(el.detachEvent){
            el.detachEvent('on' + type, el['e'+fn]);
        }
    }
};
```
