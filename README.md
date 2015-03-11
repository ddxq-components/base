Base
==========

Base 由 [arale-base](https://github.com/aralejs/base) 修改而来，提供 [Class](https://github.com/ddxq-components/class)、[Events](https://github.com/ddxq-components/events)、Attribute 和 Aspect 支持。

----------

## 使用说明

### extend `Base.extend(properties)`

基于 Base 创建子类。例子：

```js
/* pig.js */
var Base = require('base');

var Pig = Base.extend({
    attrs: {
        name: ''
    },
    talk: function() {
        alert('我是' + this.get('name'));
    }
});

module.exports = Pig;
```

继承 Base 可覆盖 initialize 构造函数， **但需要调用父类构造函数**，如 arale 的 widget 定义了组件的生命周期

```js
/* widget.js */
var Base = require('base');

var Widget = Base.extend({
    initialize: function(config) {
        Widget.superclass.initialize.call(this, config);
        this.parseElement()
        this.initProps()
        this.delegateEvents()
        this.setup()
    },
    ...
});

module.exports = Widget;
```

Base 继承和混入了一下功能，可查看文档 ：

- [Class 使用文档](https://github.com/ddxq-components/class/)
- [Events 使用文档](https://github.com/ddxq-components/events/)


### Base 与 Class 的关系

Base 是使用 `Class` 创建的一个基础类，默认混入了 `Events`、`Attribute`、`Aspect` 模块：

```js
/* base.js */
var Class = require('class');
var Events = require('events');
var Aspect = require('./aspect');
var Attribute = require('./attribute');

var Base = Class.create({
    Implements: [Events, Aspect, Attribute],

    initialize: function(config) {
        ...
    },

    ...
});

...
```


----------------


## Attribute 使用文档

---

提供基本的属性添加、获取、移除等功能。

---

## 使用说明

基于 `Base.extend` 创建的类，会自动添加上 `Attribute` 提供的功能。例子：

```js
/* panel.js */
var Base = require('base');
var $ = require('$');

var Panel = Base.extend({
    attrs: {
        element: {
            value: '#test',
            readOnly: true
        },
        color: '#fff',
        size: {
            width: 100,
            height: 100
        },
        x: 200,
        y: 200,
        xy: {
            getter: function() {
                return this.get('x') + this.get('y');
            },
            setter: function(val) {
                this.set('x', val[0]);
                this.set('y', val[1]);
            }
        }
    },

    initialize: function(config) {
        Panel.superclass.initialize.call(this, config);
        this.element = $(config.element).eq(0);
    },

    _onChangeColor: function(val) {
        this.element.css('backgroundColor', val);
    }
});

exports.Panel = Panel;
```

**在 `initialize` 方法中，调用 `superclass.initialize` 方法，就可以自动设置好实例的属性。**

```js
/* test.js */
var Panel = require('./panel').Panel;

var panel = new Panel({
    element: '#test',
    color: '#f00',
    size: {
        width: 200
    }
});

console.log(panel.get('color')); // '#f00'
console.log(panel.get('size')); // { width: 200, height: 100 }
```

在初始化时，实例中的 `_onChangeX` 方法会自动注册到 `change:x` 事件的回调队列中：

```js
/* test2.js */
var Panel = require('./panel').Panel;

var panel = new Panel({ element: '#test' });
panel.set('color', '#00f'); // this.element 的背景色自动变为 '#00f'
```

虽然在组件实例化的时候也会设置属性，但不会触发 `change:x` 事件，即不会执行 `_onChangeX`。

## API

### attrs 的设置

类定义时, 通过设置 attrs 来定义该类有哪些属性, 每个属性是通过如下方式定义的:

```js
{
    // 方式一: 直接设置默认值
    attr1: "aString",

    // 方式二: 通过对象的 value 设置默认值, 相当于方式一
    attr2: {
        value: "bString"
    },

    // 方式三: 设置 setter
    attr3: {
        value: "cString",
        // setter 会在对象调用 set() 时触发, 可以在此时做些处理,
        // 比如强制类型转换
        // 即当 obj.set('attr3', 1) 后, 会调用 setter, 转换成 '1'
        // setter 的 this 为当前实例对象
        setter: function(v) {
            return v + ""
        }
    },

    // 方式四: 设置 getter
    attr4: {
        value: 10,
        // getter 会在对象调用 get() 时触发, 同样可以在此时做些处理,
        // 比如存的是美元, 转成人民币
        // 即当 obj.get('attr4') 后, 会调用 getter
        // getter 的 this 为当前实例对象
        getter: function(v) {
            // 美元 * 汇率 = 人民币
            return v * 6.8
        }
    },

    // 方式五: readonly
    attr5: {
        value: 0,
        // 设置 readOnly 之后, 没法通过 obj.set() 的方式设置值, 即不可更改
        // 可以同时设置 getter 来调整
        // 默认 readOnly 为 false
        readOnly: true,
        getter: function() {
            return Math.ceil(this.get('panels').length / this.get('step'));
        }
    }
}

```

### set `.set(key, value, options)`

设置某个值的属性，如果有定义 setter，会先调用 setter。

#### options.silent

`silent=true` 时不会触发 change 事件。

```
var panel = new Panel({ element: '#test' });
panel.set('color', '#00f', {silent: true}); // this.element 的背景色不会改变
```

#### options.override

如果属性值为一个简单对象，默认的方式是混合，但 `override=true` 会覆盖原来的属性。

### get `.get(key)`

获取某个属性值，如果有定义 getter，会返回 getter 的返回值。

## 交流讨论

- [after / before 与 on 的含义冲突](https://github.com/aralejs/aralejs.org/issues/74)





## Aspect 使用文档
---

使用 Aspect，可以允许你在指定方法执行的前后插入特定函数。

---

## 使用说明

基于 `Base.extend` 创建的类，会自动添加上 `Aspect` 提供的功能。


### before `object.before(methodName, callback, [context])`

在 `object[methodName]` 方法执行前，先执行 `callback` 函数。

```js
var Dialog = Base.extend({
    ...

    show: function() {
        console.log(2);
        this.element.show();
    },

    ...
});

var dialog = new Dialog();

dialog.before('show', function() {
    console.log(1);
});

dialog.after('show', function() {
    console.log(3);
});

dialog.show(); // ==> 1, 2, 3
```

`callback` 函数在执行时，接收的参数与传给 `object[methodName]` 参数相同。如果传入了
`context` 参数，则 `callback` 里的 `this` 指向 `context`。

```js
var dialog = new Dialog();

dialog.before('show', function(a, b) {
    console.log(a);
    console.log(b);
});

dialog.show(1, 2); // ==> 1, 2
```

**可以在 `callback` 中 return false 来阻止原函数执行。**

```js
dialog.before('show', function() {
    console.log(1);
    return false;
});

dialog.after('show', function() {
    console.log(3);
});

dialog.show(); // ==> 1
```


### after `object.after(methodName, callback, [context])`

在 `object[methodName]` 方法执行后，再执行 `callback` 函数。

`callback` 函数在执行时，接收的参数是 `object[methodName]` 执行完成后的返回值以及传给 `object[methodName]` 的参数。如果传入了
`context` 参数，则 `callback` 里的 `this` 指向 `context`。

```
var dialog = new Dialog();

dialog.after('show', function(returned, a, b) {
	console.log(returned); // show 的返回值
    console.log(a);
    console.log(b);
});

dialog.show(1, 2); // ==> undefined, 1, 2
```

**注意**

`before` 和 `after` 是按注册的先后顺序执行的，先注册先执行。

```js
dialog.before('show', function() {
    console.log(1);
});

dialog.before('show', function() {
    console.log(2);
});

dialog.show(); // ==> 1, 2
```





