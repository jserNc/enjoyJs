---
title: Vue 中 Observer-Dep-Watcher 三者的关系
date: 2017-12-25 17:16:18
tags: vue
---

Vue 的特点之一就是双向绑定，即“数据（data）“和”视图（view）”绑定，当数据变化时会驱动视图更新，视图变化时也会驱动数据更新。本文从 Vue 源码出发，弄清 Observer-Dep-Watcher 三者之间的关系，这对理解双向绑定这个概念非常有意义。

<!-- more -->

## 发布者工厂方法 Observer

下面通过一句代码来理解 Observer：

```
observe(data, true /* 作为根 data */);
```

(1) 如果 data 不是对象就返回，只有对象才继续执行后续步骤
(2) 如果 data 有对应的 Observer 实例 data.__ob__ 那就将它作为 observe 方法返回值
(3) 如果 data 没有对应的 Observer 实例，那就执行 ob = new Observer(value)
(4) new Observer(value) 的本质是执行 ob.walk(data)
(5) 依次遍历 data 的属性 key，执行 defineReactive$$1(obj, keys[i], obj[keys[i]])
(6) defineReactive$$1 会劫持属性 key 的 get/set 操作。
(7) 当获取属性 key 时除了返回属性值，还会将 Dep.target（即与属性 key 对应的 watcher）加入到 key 的订阅者数组里（dep.depend() -> Dep.target.addDep(dep)）
(8) 当设置属性 key 时除了更新属性值外，还会由主题对象 dep 发出通知给所有的订阅者 dep.notify()

总的来说就是：observe(data) -> new Observer(data) -> defineReactive$$1()

下面着重看一下 defineReactive$$ 方法：

```
// 在 obj 对象上劫持 key 属性的 get/set 操作
function defineReactive$$1(obj,key,val,customSetter,shallow){

  // 新建一个主题对象
  var dep = new Dep();
  
  /*
  对象的每一个属性都有相应的描述对象，例如：

  var o = {a:1}
  var props = Object.getOwnPropertyDescriptor(o,'a')
  -> { 
      value: 1, 
      writable: true, 
      enumerable: true, 
      configurable: true
    }
  */
  var property = Object.getOwnPropertyDescriptor(obj, key);

  // 如果 obj 的 key 属性不可配置，直接返回
  if (property && property.configurable === false) {
    return
  }

  // 之前定义的 getter/setters
  var getter = property && property.get;
  var setter = property && property.set;

  /*
   shallow 的意思是"浅的"，也就是说没有指定"浅观察"，就是深度观察

   举例来说：
   obj = {
     a : {
        aa : 1
     },
     b : {
        bb : 2
     }
   }

   这里的 val 就不同普通的原始类型值了，val 是 {aa : 1}，{bb : 1} 
   这样的对象，那么就递归遍历 val 对象的属性，劫持其属性的 gett/set

  */
  
  // 【重要】这句作用就是：递归遍历劫持 val 的所有子属性
  var childOb = !shallow && observe(val);

  /*
    Object.defineProperty(obj, prop, descriptor) 
    参数：
    obj 需要定义属性的对象。
    prop 需被定义或修改的属性名。
    descriptor 需被定义或修改的属性的描述符

    作用：直接在一个对象上定义一个新属性，或者修改一个已经存在的属性
   */
  Object.defineProperty(obj, key, {
    // 可枚举
    enumerable: true,
    // 可配置
    configurable: true,
    // 获取 obj 的 key 属性时触发该方法
    get: function reactiveGetter () {
      var value = getter ? getter.call(obj) : val;
      if (Dep.target) {
        // 相当于 Dep.target.addDep(dep)
        dep.depend();
        if (childOb) {
          // 相当于 Dep.target.addDep(childOb.dep)
          childOb.dep.depend();
        }
        if (Array.isArray(value)) {
          // 对数组每项 e 调用 Dep.target.addDep(e.__ob__.dep)
          dependArray(value);
        }
      }
      return value
    },
    // 设置 obj 的 key 属性时触发该函数
    set: function reactiveSetter (newVal) {
      // 获取旧值
      var value = getter ? getter.call(obj) : val;

      // 如果旧值和新值相等或者旧值和新值都是 NaN，则不进行设置操作。
      //（NaN 是唯一不等于自身的值）
      if (newVal === value || 
         (newVal !== newVal && value !== value)) {
        return
      }

      // 执行自定义 setter
      if ("development" !== 'production' && customSetter) {
        customSetter();
      }
    
      // 设置新值
      if (setter) {
        setter.call(obj, newVal);
      } else {
        // 注意：set/set 函数在这里是闭包，所以能共用 val 的值
        val = newVal;
      }
      
      // 递归遍历 newVal 的所有子属性
      childOb = !shallow && observe(newVal);

      // 发通知订阅者，分别执行 update 方法
      dep.notify();
    }
  });
}
```

## 发布者-订阅者之间的主题工厂方法 Dep

Dep 的作用是创建主题。主题的作用是替发布者管理订阅者。每一个发布者可以有多个订阅者，主题可以对订阅者进行增加、删除等操作。最重要的是当发布者有新的动作是，主题还可以把消息传达给所有订阅者，订阅者就会执行特定方法（这里是 update 方法）来完成相应的操作了。

```
var uid = 0;

// 主题工厂方法，Dep 是 dependency 的简写，也可以叫依赖
var Dep = function Dep () {
  this.id = uid++;
  // 订阅者将被添加到这个数组里
  this.subs = [];
};

// 添加订阅者（watcher）
Dep.prototype.addSub = function addSub (sub) {
  this.subs.push(sub);
};

// 删除订阅者
Dep.prototype.removeSub = function removeSub (sub) {
  // 先找到 sub 在 this.subs 中的索引，然后删除它
  remove(this.subs, sub);
};

// 添加依赖
Dep.prototype.depend = function depend () {
  if (Dep.target) {
    // 相当于 this.addSub(Dep.target)
    // 其中 Dep.target 为订阅者（watcher）
    Dep.target.addDep(this);
  }
};

// 触发更新
Dep.prototype.notify = function notify () {
  // 先复制一份订阅者数组，以免执行下面 for 循环过程中该数组变化了
  var subs = this.subs.slice();
  for (var i = 0, l = subs.length; i < l; i++) {
    subs[i].update();
  }
};

// 任何时候只有一个全局唯一的正在进行计算的订阅者
Dep.target = null;

var targetStack = [];

// 旧的 Dep.target 压栈，_target 作为新的 Dep.target
function pushTarget (_target) {
  if (Dep.target) { targetStack.push(Dep.target); }
  Dep.target = _target;
}

// Dep.target 出栈，即恢复旧的 Dep.target
function popTarget () {
  Dep.target = targetStack.pop();
}
``` 

## 订阅者工厂方法 Watcher

每一个 Watcher 实例就是一个订阅者，为什么订阅者可以起到观察某个 expOrFn（表达式/函数）的作用呢？这里分两种情况说明：

① expOrFn 为表达式，例如 'a'

```
// 新建 Vue 实例
vm = new Vue({
  data : {
    aaa : {
        bbb : {
            ccc : 1
        }
    }
  }
});

// 新建 watcher
var watcher = new Watcher(vm, 'aaa.bbb.ccc' , cb, options);
```

理一理这个 watcher 工作的基本流程：

(1) 执行 watcher = new Watcher() 会定义 watcher.getter = parsePath('aaa.bbb.ccc')（这是一个函数，稍后会解释），同时也会定义 watcher.value = watcher.get()，而这会触发执行 watcher.get()
(2) 执行 watcher.get() 就是执行 watcher.getter.call(vm, vm)，也就是 parsePath('aaa.bbb.ccc').call(vm, vm)
(3) 执行 parsePath('aaa.bbb.ccc').call(vm, vm) 会触发 vm.aaa.bbb.ccc 属性读取操作
(5) vm.aaa.bbb.ccc 属性读取会触发 aaa.bbb.cc 属性的 get 函数（在 defineReactive$$1 函数中定义）
(6) get 函数会触发 dep.depend()，也就是 Dep.target.addDep(dep)，即把 Dep.target 这个 Watcher 实例添加到 dep.subs 数组里（也就是说，dep 可以发布消息通知给订阅者 Dep.target）
(7) 那么 Dep.targe 又是什么呢？其实 (2) 中执行 watcher.get() 之前已经将 Dep.target 锁定为当前 watcher（等到 watcher.get() 执行结束时释放 Dep.target）
(8) 于是，watcher 就进入了 aaa.bbb.ccc 属性的订阅数组，也就是说 watcher 这个订阅者订阅了 aaa.bbb.ccc 属性
(9) 当给 aaa.bbb.ccc 属性赋值时，如 vm.aaa.bbb.ccc = 100 会触发 vm 的 aaa.bbb.ccc 属性的 set 函数（在 defineReactive$$1 函数中定义）
(10) set 函数触发 dep.notify()
(11) 执行 dep.notify() 就会遍历 dep.subs 中的所有 watcher，并依次执行 watcher.update()
(12) 执行 watcher.update() 又会触发 watcher.run()
(13) watcher.run() 触发 watcher.cb.call(watcher.vm, value, oldValue);

结合 parsePath 函数的定义，理解一下 parsePath('aaa.bbb.ccc')

```
// 在这里
path = 'aaa.bbb.ccc' 

segments = path.split('.')
-> segments = ["aaa", "bbb", "ccc"]

// 于是
watcher.getter = parsePath('aaa.bbb.ccc') = function (obj) {
  for (var i = 0; i < segments.length; i++) {
    if (!obj) { return }
    obj = obj[segments[i]];
  }
  return obj
};

// 所以
watcher.getter.call(vm, vm)
-> parsePath('aaa.bbb.ccc')(vm)
-> vm.aaa.bbb.ccc

// 也就是说这会触发 vm 的属性获取操作
```

② expOrFn 为函数，例如 updateComponent（该函数作用是更新视图）

```
vm._watcher = new Watcher(vm, updateComponent, noop);
```

理一理这个 watcher 工作的基本流程：

(1) 执行 vm._watcher = new Watcher() 会定义 vm._watcher.getter = updateComponent，同时也会定义 vm._watcher.value = vm._watcher.get()，而这会触发执行 vm._watcher.get()
(2) 执行 vm._watcher.get() 就是执行 vm._watcher.getter.call(vm, vm)，也就是 updateComponent(vm, vm) 即可进行视图更新（事实上，视图的初始化更新就是这么完成的）

若要要再次主动更新视图怎么做？

(1) Vue.prototype.$forceUpdate() 触发 vm._watcher.update()
(2) vm._watcher.update() 触发 vm._watcher.run()
(3) vm._watcher.run() 执行 value = this.get()，触发 vm._watcher.get()，也就是执行 updateComponent(vm, vm)，这会导致视图再次更新

着重看一下 Watcher.prototype.run 方法：

```
// 第 1 步，获取最新的值（这会导致 this.getter 方法执行）
var value = this.get();

// 第 2 步，新值和旧值对比，不同则继续第 3 步
value !== (oldValue = this.value)

// 第 3 步，执行 cb 回调函数，参数分别为新值和旧值
this.cb.call(this.vm, value, oldValue);
```

最后，用一张图做个总结：

![foo对象结构](/css/images/observer-dep-watcher/observer-dep-watcher.png)



参考：
[1] https://github.com/bison1994
[2] https://www.cnblogs.com/kidney/p/6052935.html
[3] https://www.cnblogs.com/libin-1/p/6893712.html
