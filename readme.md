## Vue源码解析

### 基本概念知识

- 将伪数组转换成真数组：`[].slice.call(fakeArray)`

```javascript
// call的源码实现
Function.prototype.myCall = function(context){
    context = context || window
    //第一步获取调用myCall的方法
    context.fn = this
    //第二步剔除myCall第一个参数
    var temp = []
    for(var i=1;i<arguments.length;i++){
        temp.push(`arguments[${i}]`)
    }
    //第三步执行调用myCall函数
    eval(`context.fn(${temp})`)
}
// slice的源码实现
Array.prototype.mySlice = function(){
    var start = arguments[0] || 0
    var end = arguments[1] || this.length
    var temp = []
    for(var i=start;i<end;i++){
        temp.push(this[i])
    }
    return temp
}
```

- 得到节点类型：`node.nodeType`

[官方文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/nodeType)

```html
<div id='test'>hello world</div>
<script>
var elementNode = document.getElementById('test')
var attrNode = elementNode.getAttributeNode('id')
var textNode = elementNode.firstChild
console.log(elementNode.nodeType,attrNode.nodeType,textNode.nodeType)
</script>
```



- 给对象添加属性（指定描述符）：`Object.defineProperty(obj, propertyName,{})`

```javascript
var book = {
    __year:2004,
    edition:1
}
Object.defineProperty(book,'year',{
    get:function(){
        return this.__year
    },
    set:function(newValue){
        if(newValue>2004){
            this.__year = newValue
            this.edition += newValue - 2004
        }
    }
})
book.year = 2005
console.log(book.edition)
```

- 得到对象自身可枚举属性组成的数组：`Object.keys(obj)`

```javasc
function Person(){
    Person.prototype.name = 'Nicholas'
    Person.prototype.age = 20
    Person.prototype.job = 'software Engineer'
}
var keys = Object.keys(Person.prototype)
console.log(keys)  //name,age,job

var p1 = new Person()
p1.name = 'Rob'
p1.age = 31
var p1key = Object.keys(p1)
console.log(p1keys)
```



- 判断prop是obj自身属性还是来源于原型：`obj.hasOwnProperty(prop)`

```javascript
function Person(){
    Person.prototype.name = 'Nicholas'
}
var person1 = new Person()
console.log(person01.hasOwnProperty('name'))  //fasle

person1.name = 'Greg'
console.log(person01.hasOwnProperty('name'))  //true
```



- 文档碎片（高校批量更新多个节点）：`DocumentFragment`

不同于Document，Document中的元素进行更新的时候，浏览器渲染引擎会重新进行渲染工作。而DocumentFragement是存储在内存中承载元素节点的容器，该容器上的更新并不会导致浏览器引擎进行渲染。这样导致的结果便是，通过DocumentFragment可以减少JS引擎和渲染引擎的交流次数。

原因在于JS引擎与渲染引擎每次交流都会产生额外开销。



### 数据代理

- 通过一个对象代理对另一个对象中属性的操作（读/写）
- vue数据代理：通过vm对象来代理data对象中所有属性的操作
- 好处：更方便的操作data中的数据

实现原理：

- 将构造函数中的配置对象赋值给实例
- 通过Object.keys()遍历实例属性对象上的属性
- 通过Object.defineproperty(obj,property,{})设置属性的读写，从而实现数据代理

```javascript
function MVVM(options){
    this.$options = options ||
    var data = this._data = this.$options.$data
    var obj = this
    Object.keys(data).forEach(item => {
        obj.__proxyData(item)
    })
}
MVVM.prototype = {
    var _this = this
    Object.defineProperty(_this,item,{
    	configurable:false,
    	enumerable:true,
    	get(){
    		return _this._data.item
		},
    	set(newVuale){
    		_this._data.item = newValue
		}
	})
}
```

### 模板编译

- 将页面DOM节点放入到内存的fragment当中
- 编译模板：将插值表达式，vm属性内容正确解析
  - 重难点
- 将编译好的fragment再重新插入到页面当中

### 数据绑定

