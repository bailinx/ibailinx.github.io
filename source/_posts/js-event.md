# 深入理解JavaScript事件系统
之前研究重写alert/confirm时，曾遇到一问题：js如何绑定和销毁匿名事件。怀着这个问题，一起来研究一下js的事件系统。

## DOM0事件模型
DOM0事件模型就是直接在Dom对象上注册事件名称，所有浏览器均支持，形如
'''javascript
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
// 不过需要注意的是，IE6/7/8下可能会造成内存泄漏，正确的方法是用完即删除
btn.onclick = null;

'''
