---
title: Vue 2.x 全局 api 详解
date: 2017-12-19 09:34:36
tags: vue
---

vue 官网上专门列出了全局 api 供大家参考，这些 api 我看了不下几遍，但是对其用法始终没能理解得很透彻。于是抽空了解了一下 vue 2.4.0 的源码，这里对这些全局 api 做一些尽可能详细的注解。

<!-- more -->

[Vue.extend](#vue-extend)
[Vue.nextTick](#vue-nextTick)
[Vue.set](#vue-set)
[Vue.delete](#vue-delete)
[Vue.directive](#vue-directive)
[Vue.filter](#vue-filter)
[Vue.component](#vue-component)
[Vue.use](#vue-use)
[Vue.mixin](#vue-mixin)
[Vue.compile](#vue-compile)
[Vue.version](#vue-version)


<h2 id="vue-extend">Vue.extend( options )</h2>

使用基础 Vue 构造器，创建一个“子类”。注释后的源码如下：

```
Vue.extend = function (extendOptions) {
    extendOptions = extendOptions || {};

    // 父类
    var Super = this;
    var SuperId = Super.cid;

    /*
      cachedCtors 缓存父类 cid - 子类集合，结构为：
      {
        SuperId1 ： Sub1,
        SuperId2 ： Sub2,
        ...
      }
     */
    var cachedCtors = 
    extendOptions._Ctor || (extendOptions._Ctor = {});

    // 如果能从缓存取到子类，就省去了下面的所有步骤
    if (cachedCtors[SuperId]) {
      return cachedCtors[SuperId]
    }
    
    // 组件名
    var name = extendOptions.name || Super.options.name;
    { 
      // 提示：有效的组件名之内包含字母数字和连字符，并且要以字母开头
    }

    // 子类
    var Sub = function VueComponent (options) {
      this._init(options);
    };

    // 子类继承父类原型，并修正 constructor 属性指向
    Sub.prototype = Object.create(Super.prototype);
    Sub.prototype.constructor = Sub;

    // 每个类都有唯一的 cid
    Sub.cid = cid++;

    // 根据合并策略，合并 options
    Sub.options = mergeOptions(
      Super.options,
      extendOptions
    );

    // super 属性指向父类
    Sub['super'] = Super;

    if (Sub.options.props) {
      initProps$1(Sub);
    }
    if (Sub.options.computed) {
      initComputed$1(Sub);
    }

    // 获取父类的 extend、mixin 和 use 方法
    Sub.extend = Super.extend;
    Sub.mixin = Super.mixin;
    Sub.use = Super.use;

    /*
    // 配置类型
    var ASSET_TYPES = [
      'component',
      'directive',
      'filter'
    ];
    */
    ASSET_TYPES.forEach(function (type) {
      Sub[type] = Super[type];
    });


    if (name) {
      Sub.options.components[name] = Sub;
    }

    Sub.superOptions = Super.options;
    Sub.extendOptions = extendOptions;
    Sub.sealedOptions = extend({}, Sub.options);

    // cache constructor，缓存构造函数
    cachedCtors[SuperId] = Sub;
    return Sub
  };
}
```

归纳一下：
① 对于每一个扩展选项对象 extendOptions，为其添加一个 extendOptions._Ctor 属性，该属性是 "父类 cid - 子类" 的集合，也就是对子类的缓存。对于同一个 extendOptions 和同一个父类可以唯一确定一个子类，只要子类被生成过，下次就可以从缓存中取了。
② 对组件名单独做判断，有效的组件名之内包含字母数字和连字符，并且要以字母开头。
③ 定义子类构造函数，让子类继承父类的原型方法，并将父类的一系列静态方法也赋给子类。
④ 将当前生成的子类缓存到 extendOptions._Ctor 中，并返回子类。

所以，以下两种写法可以达到同样的效果：

```
// html
<div id="mount-point"></div>

// 写法一：
// 创建 Vue 实例
var vm = new Vue({
  template: '<p>{{ title }}</p>',
  data: function () {
    return {
      title : 'enjoy javascrip',
    }
  }
})
// 挂载到一个元素上。
vm.$mount('#mount-point')

// 写法二：
// 创建子类
var Profile = Vue.extend({
  template: '<p>{{ title }}</p>',
  data: function () {
    return {
      title : 'enjoy javascrip',
    }
  }
})
// 创建 Profile 实例，并挂载到一个元素上。
new Profile().$mount('#mount-point')
```

<h2 id="vue-nextTick">Vue.nextTick( [callback, context] )</h2>

本轮“事件循环”结束后执行 callback 函数。注释后的源码如下：

```
// nextTick 方法挂载到 Vue 对象上，成为全局方法
Vue.nextTick = nextTick;

var nextTick = (function () {
  var callbacks = [];
  var pending = false;
  var timerFunc;

  function nextTickHandler () {
    pending = false;
    // 深拷贝
    var copies = callbacks.slice(0);
    // 将 callbacks 数组清空
    callbacks.length = 0;
    // 依次执行原 callbacks 数组里的方法
    for (var i = 0; i < copies.length; i++) {
      copies[i]();
    }
  }

  // ① 原生支持 Promise 时，用 Promise 触发 nextTickHandler
  if (typeof Promise !== 'undefined' && isNative(Promise)) {
    var p = Promise.resolve();

    var logError = function (err) { console.error(err); };
    timerFunc = function () {
      p.then(nextTickHandler).catch(logError);
      if (isIOS) { setTimeout(noop); }
    };
  // ② 否则，用 MutationObserver 来触发 nextTickHandler
  } else if (typeof MutationObserver !== 'undefined' && (
    isNative(MutationObserver) ||
    MutationObserver.toString() === 
    '[object MutationObserverConstructor]'
  )) {
    var counter = 1;
    // 指定 observer 的回调方法是 nextTickHandler
    var observer = new MutationObserver(nextTickHandler);
    // observer 的作用是监视 dom 变化，所以这里创建一个 dom
    var textNode = document.createTextNode(String(counter));
    // 监视 textNode
    observer.observe(textNode, {
      characterData: true
    });
    timerFunc = function () {
      // counter 变化 -> dom 变化 -> 触发 nextTickHandler()
      counter = (counter + 1) % 2;
      textNode.data = String(counter);
    };
  // ③ 前两者都不支持，用 setTimeout 来模拟异步任务
  } else {
    timerFunc = function () {
      setTimeout(nextTickHandler, 0);
    };
  }

  return function queueNextTick (cb, ctx) {
    var _resolve;
    // 将 cb 分别用一个新的匿名函数包装，并 push 进 callbacks 数组
    callbacks.push(function () {
      if (cb) {
        try {
          cb.call(ctx);
        } catch (e) {
          handleError(e, ctx, 'nextTick');
        }
      } else if (_resolve) {
        _resolve(ctx);
      }
    });
    // pending 状态改为 true
    if (!pending) {
      pending = true;
      timerFunc();
    }

    if (!cb && typeof Promise !== 'undefined') {
      return new Promise(function (resolve, reject) {
        _resolve = resolve;
      })
    }
  }
})();

```

虽然 Promise 和 MutationObserver 都可以在“事件循环”结束后异步触发回调函数，但由于 MutationObserver 在 ios 系统有 bug，所以还是首选还是 Promise。

**（1）先来看看 MutationObserver：**

MutationObserver（变动观察器）是监视 DOM 变动的接口。当 DOM 对象树发生任何变动时，MutationObserver 会得到通知。在概念上，它很接近事件。可以理解为，当 DOM 发生变动会触发 MutationObserver 事件。但是，它与事件有一个本质不同：

事件是同步触发，也就是说 DOM 发生变动立刻会触发相应的事件；而 MutationObserver 则是异步触发，DOM 发生变动以后，并不会马上触发，而是要等到当前所有 DOM 操作都结束后才触发。ß

这样设计是为了应付 DOM 变动频繁的情况。举例来说，如果在文档中连续插入 1000 个段落（p 元素），会连续触发 1000 个插入事件，执行每个事件的回调函数，这很可能造成浏览器的卡顿；而 Mutation Observer 完全不同，只在 1000 个段落都插入结束后才会触发，而且只触发一次。

**（2）理解一下 Promise 中的 resolve 方法：**

```
var _resolve;
var p = new Promise(function (resolve, reject) {
    _resolve = resolve;
})
p.then(function(data){
    console.log(data);
})

_resolve('这是数据')
// 控制台打印'这是数据'
```

resolve 函数的作用是，将 Promise 对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved）。而 then 方法可以接受两个回调函数作为参数。第一个回调函数是 Promise 对象的状态变为 resolved 时调用。所以，执行 _resolve 函数后，就会执行 then 方法指定的第一个回调，并且 _resolve 的实参会传给 then 的第一个回调函数。
    

简化一下 nextTick 函数代码流程：

```
var callbacks = [];
var pending = false;
var timerFunc;

function nextTickHandler () {
    // 依次执行 callbacks 队列中的函数，并清空该队列，解锁。
}


timerFunc = function () {
    // 异步执行 nextTickHandler()
};

var nextTick = function queueNextTick (cb, ctx) {
    var _resolve;

    // 将匿名函数封装 cb/_resolve，并推入 callbacks 队列
    callbacks.push(function () {
      if (cb) {
        cb.call(ctx);
      } else if (_resolve) {
        _resolve(ctx);
      }
    });
    
    // 执行 timerFunc() 并上锁
    if (!pending) {
      pending = true;
      timerFunc();
    }

    // 参数为空并且不支持 Promise
    if (!cb && typeof Promise !== 'undefined') {
      return new Promise(function (resolve, reject) {
        // 于是可以用 _resolve 方法来触发 Promise 实例的 then 回调
        _resolve = resolve;
      })
    }
}

```


那么，若条件 !cb && typeof Promise !== 'undefined' 满足，理解一下以下代码：

```
// nextTick() 不带参数时，可以当做 Promise 实例
var p = nextTick();

p.then(function(data){
    console.log(data);
})
```
  
则本轮“事件循环”结束后会执行 nextTickHandler 函数，也就是说会依次触发队列 callbacks 中的函数，其中包括 _resolve(ctx)。而一旦执行了 _resolve(ctx)，就会执行 then 的第一个回调函数，即打印出 ctx。

总结一下 nextTick 函数的用法：

```
var nextTick = function queueNextTick (cb, ctx) {...}
```

a) 若 cb 参数不存在或当前环境不支持 Promise，则没有指定返回值，也就是 undefined；
b) 否则，返回一个 promise 实例。

简单点说就是：
① nextTick 方法有实参时，将实参加入回调函数队列 callbacks，然后在本轮“事件循环”结束后，依次执行回调队列 callbacks 中的函数；
② nextTick 方法没有实参时，返回一个 Promise 实例。可以为该实例添加 then 回调，待队列 callbacks 中函数执行 _resolve(ctx) 时触发 then 的回调方法
   
<h2 id="vue-set">Vue.set( target, key, value )</h2>

设置对象的属性。如果对象是响应式的，确保属性被创建后也是响应式的，同时触发视图更新。

```
// set 方法挂载到 Vue 对象上，成为全局方法
Vue.set = set;

function set (target, key, val) {

  // ① target 是数组，并且 key 是合法的数组索引
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    // 数组长度变为 target.length 和 key 中的较大者
    target.length = Math.max(target.length, key);
    // 在 key 处删除 1 个元素，并新增 val 元素，其实就是替换（设置）
    target.splice(key, 1, val);
    // 返回
    return val
  }

  // ② 如果 key 是 target 对象的自身属性，直接赋值，返回
  if (hasOwn(target, key)) {
    target[key] = val;
    return val
  }

  // Observer 实例
  var ob = (target).__ob__;

  // target 对象是 Vue 实例，或者 Vue 实例的根数据对象
  if (target._isVue || (ob && ob.vmCount)) {
    // 非生产环境，发出不能设置的警告，并返回
    return val;
  }

  // 普通对象，直接赋值，返回
  if (!ob) {
    target[key] = val;
    return val
  }

  /*
    走到这说明 ob 存在，即对象 target 是响应式的

    ob.value 其实就是 target。以下几句作用是：
    如果对象是响应式的，那么新添加的属性也是响应式的，并触发视图更新
   */
  defineReactive$$1(ob.value, key, val);
  ob.dep.notify();
  return val
}
```

<h2 id="vue-delete">Vue.delete( target, key )</h2>

删除对象的属性。如果对象是响应式的，确保删除能触发更新视图。

```
// del 方法挂载到 Vue 对象上，成为全局方法
Vue.delete = del;

function del (target, key) {
  // target 是数组，并且 key 是合法的数组索引，直接删除 key 处元素
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.splice(key, 1);
    return
  }

  var ob = (target).__ob__;

  // target 对象是 Vue 实例，或者 Vue 实例的根数据对象
  if (target._isVue || (ob && ob.vmCount)) {
    // 非生产环境，发出不能删除的警告，并返回
    return
  }

  // 非自身属性，直接返回。也就是说只能删除自身属性。
  if (!hasOwn(target, key)) {
    return
  }

  // 删除自身属性
  delete target[key];

  if (!ob) {
    return
  }
  // ob 存在，也就是说 target 是响应式对象，则更新视图
  ob.dep.notify();
}
```


<h2>
    <span id="vue-directive">Vue.directive( id, [definition] )</span>
    <span id="vue-filter">Vue.filter( id, [definition] )</span>
    <span id="vue-component">Vue.component( id, [definition] )</span>
</h2>

之所以将这三个方法放在一起，是因为它们是一起定义的。其中 Vue.directive 注册或获取全局指令。Vue.filter 注册或获取全局过滤器。Vue.component 注册或获取全局组件。三个函数作用很类似，一个参数表示获取，两个参数表示注册。

```
var ASSET_TYPES = [
  'component',
  'directive',
  'filter'
];

ASSET_TYPES.forEach(function (type) {
  // 对 definition 进行修正，最后返回 definition
  Vue[type] = function (id, definition) {

    // ① 只有一个实参就是【获取】注册的组件
    if (!definition) {
      return this.options[type + 's'][id]
    // ② 两个参数，注册组件。3 种特殊处理。
    } else {

      // 特殊处理一：Vue.component 的参数 id
      if(type === 'component' && config.isReservedTag(id)){
        // 警告：id 不能是保留标签名
      }

      // 特殊处理二：Vue.component 的参数 definition
      if(type === 'component' && isPlainObject(definition)){
        definition.name = definition.name || id;
        /*
          其中 Vue.options._base = Vue
          所以 definition = Vue.extend(definition)，也就是说：

          如果 definition 是普通对象，自动调用 Vue.extend 修正它

          所以：Vue.component('my-com', {...})
          实质是：Vue.component('my-com', Vue.extend({...}))
         */
        definition = this.options._base.extend(definition);
      }
    
      // 特殊处理三：Vue.directive 的参数 definition
      if(type === 'directive' && 
         typeof definition === 'function'){
        /*
          如果 definition 是函数，那就将它修正为对象：
          { bind: definition, update: definition }

          也就是说以下两种写法等价：
          Vue.directive('my-directive', myFunc)
          
          Vue.directive('my-directive', {
            bind: myFunc,
            update: myFunc
          })
         */
        definition = {bind: definition, update: definition};
      }

      // 注册组件，然后返回。注意 type 后跟 's'
      this.options[type + 's'][id] = definition;
      return definition
    }
  };
});
```

理解一下注册全局组件的方式 Vue.component(tagName, options)：

```
Vue.component('my-component', {
  // 选项列表
})

// 相当于：
Vue.component('my-component', Vue.extend({
  // 选项列表
})

// 还相当于：
this.options['components']['my-component'] = Vue.extend({
  // 选项列表
}

// 于是，自定义元素 <my-component></my-component> 
// 就可以在 Vue 的实例的模板中使用了
```

<h2 id="vue-use">Vue.use( plugin )</h2>

安装 Vue.js 的插件。

```
Vue.use = function (plugin) {

  // 已经安装过的插件
  var installedPlugins = 
  (this._installedPlugins || (this._installedPlugins = []));
  // 如果当前插件已经安装过，不重复安装，直接返回
  if (installedPlugins.indexOf(plugin) > -1) {
    return this
  }

  // 第一个参数是 plugin，其余参数作为 plugin 执行时的实参
  var args = toArray(arguments, 1);

  // 将 this 也作为 plugin 执行时的实参
  args.unshift(this);

  // ① plugin 是对象，那就执行 plugin.install 函数
  if (typeof plugin.install === 'function') {
    plugin.install.apply(plugin, args);
  // ② plugin 是函数，那直接执行 plugin 函数
  } else if (typeof plugin === 'function') {
    plugin.apply(null, args);
  }

  // 将当前插件加入已安装过的插件数组里
  installedPlugins.push(plugin);
  return this
};
```

我们可以用在插件中添加全局方法或属性、添加全局资源、注册组件、添加实例方法等等。具体用法一般分为以下几步：

```
(1) 开发插件
MyPlugin.install = function (Vue, options) {
  // 添加一个全局方法
  Vue.myFunc = function () {
    console.log('这是插件定义的全局方法')
  }
}

(2) 安装插件
Vue.use(MyPlugin)


(3) 使用插件
Vue.myFunc()
```

<h2 id="vue-mixin">Vue.mixin( mixin )</h2>

全局注册一个混合，影响注册之后所有创建的每个 Vue 实例。

```
Vue.mixin = function (mixin) {
  this.options = mergeOptions(this.options, mixin);
  return this
};
```

由于 Vue.mixin() 改变的是 Vue.options。所以这会影响之后所有创建的每个 Vue 实例。看一个官网实例：

```
// 为自定义的选项 'myOption' 注入一个处理器。
Vue.mixin({
  // Vue.prototype._init 执行完各项初始化后会执行 created 钩子函数
  created: function () {
    /*
      this.$options 指创建 Vue 实例时的配置对象，如下面的
      {
        myOption: 'hello!'
      }
     */
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})

// 新建 Vue 实例会执行 Vue.prototype._init 函数
new Vue({
  myOption: 'hello!'
})
// => "hello!"
```

关于配置对象，看看这个很关键的 mergeOptions 函数：

```
// 合并 options 对象
function mergeOptions (parent, child, vm) {
  { 
    // 打印出 child.components 中不符合要求的组件名
    checkComponents(child);
  }

  // 对 child 进行修正
  if (typeof child === 'function') {
    child = child.options;
  }

  // 对 child 进行 3 种格式化
  // ① 将 child.props 的每一项都统一成对象格式
  normalizeProps(child);
  // ② 将 options.inject 转化为对象格式
  normalizeInject(child);
  // ③ 将 child.directives 的每一项都统一成对象格式
  normalizeDirectives(child);


  // 对 parent 进行 2 种修正
  // ① 用 child.extends 修正 parent
  var extendsFrom = child.extends;
  if (extendsFrom) {
    parent = mergeOptions(parent, extendsFrom, vm);
  }
  // ② 用 child.mixins 的每一项修正 parent
  if (child.mixins) {
    for (var i = 0, l = child.mixins.length; i < l; i++) {
      parent = mergeOptions(parent, child.mixins[i], vm);
    }
  }

  var options = {};
  var key;

  // 遍历 parent 的所有属性
  for (key in parent) {
    mergeField(key);
  }
  // 遍历 child 中所有属性（排除 parent 中已有属性）
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key);
    }
  }

  // 从这里可以看出 options 是 parent 和 child 的并集

  // 合并 key 属性
  function mergeField (key) {
    /*
    ① strats = config.optionMergeStrategies 
       我们可以为该对象添加方法属性，自定义合并策略的选项

       每一个 strats[key] 是一个 functio
       不同的 key 对应不同的 function，也就是不同的合并策略

    ② defaultStrat(parentVal, childVal) 作为默认的合并函数
       作用为：
       a) 只要 childVal 不是 undefined，那就返回 childVal
       b) childVal 全等于 undefined，返回 parentVal

    如果没有对某个 key 属性指定合并策略，就用默认的策略 defaultStrat
    */
    var strat = strats[key] || defaultStrat;
    options[key] = strat(parent[key], child[key], vm, key);
  }
  // 返回合并后的选项对象
  return options
}
```

这里提到合并策略，那就多说几句，以便理解这个概念。

```
// (1) 首先，定义了一个合并策略对象：
var config.optionMergeStrategies = Object.create(null);
var strats = config.optionMergeStrategies;

// (2) 逐步向这个空的对象添加属性，每个属性对应一个合并策略函数，例如：
// ① strats.el 、strats.propsData
strats.el = strats.propsData = function(parent,child,vm,key){
  ...
  return defaultStrat(parent, child)
};

// ② strats.data
strats.data = function (parentVal,childVal,vm) {...}

// ③ strats.beforeCreate、strats.created ...
var LIFECYCLE_HOOKS = [
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeUpdate',
  'updated',
  'beforeDestroy',
  'destroyed',
  'activated',
  'deactivated'
];

LIFECYCLE_HOOKS.forEach(function (hook) {
    strats[hook] = mergeHook;
});

function mergeHook (parentVal,childVal) {...}

// 后面还有其他属性也是这样添加的，这里就不一一列举了
// 可以看到，每一个 strats[key] 函数形式都很类似
// 第一个参数是 parentVal，第二个参数是 childVal ...
// 每个 strats[key] 函数的作用就是定义该 key 属性的合并策略
// 不同的属性，采取不同的策略，自由度比较大，方便扩展

// (3) 在 mergeField 函数中调用 strats[key] 函数来合并单个选项
function mergeField (key) {
  var strat = strats[key] || defaultStrat;
  options[key] = strat(parent[key], child[key], vm, key);
}

// 其中 defaultStrat(parentVal, childVal) 作为默认的合并函数
```

<h2 id="vue-compile">Vue.compile( template )</h2>

将模板字符串转为真正的渲染函数。

```
Vue$3.compile = compileToFunctions;

```

其实，这里的 Vue$3 就是 Vue，看一下整个 Vue 框架结构就明白了：

```
(function (global, factory) {
  // CommonJS 规范
  typeof exports==='object' && typeof module!=='undefined'? 
  module.exports = factory() :
  // AMD 规范
  typeof define==='function' && define.amd ? define(factory):
  (global.Vue = factory());
}(this, (function () {
  // 真正的构造函数
  function Vue$3 (options) {
    // ...
  }
  ...
  return Vue$3;
})));

// 在浏览器环境下 exports 和 define 默认都是未定义，简化一下代码：
(function (global, factory) {
  global.Vue = factory();
}(this, (function () {
  ...
  return Vue$3;
})));

// 加上打印语句以查看代码执行流程：
(function (global, factory) {
  console.log('给 Vue 赋值');
  global.Vue = factory();
  console.log('Vue 的值: ', Vue);
}(this, (function () {
  console.log('执行 factory 函数');
  return 'enjoyJs';
})));

/*
打印结果：
① 给 Vue 赋值
② 执行 factory 函数
③ Vue 的值:  enjoyJs
*/

// 于是，Vue 和 Vue$3 的关系一目了然
global.Vue = Vue$3;

// 这里的 global 就是 this，在浏览器环境下就是 window
// 挂载在 window 上的变量就是全局的，即 window.Vue 和 Vue 是一回事
```

再回到 compileToFunctions 函数：

```
function compileToFunctions (template, options, vm) {
  options = options || {};

  {
    // CSP（Content Security Policy）检测，一旦报错，发出警告
  }

  // 由于同一个 template 可以搭配不同的 delimiters
  // 所以 delimiters + template 才能唯一标志一个模板
  var key = options.delimiters
    ? String(options.delimiters) + template
    : template;

  // 首先从缓存去取，取到了就返回
  if (cache[key]) {
    return cache[key]
  }

  // 第 1 步：编译
  var compiled = compile(template, options);
  /*
       compiled 结构:
       {
          ast: ast,
          render: code.render,
          staticRenderFns: code.staticRenderFns
          errors: [...],
          tips: [...]
        }
   */

  // check compilation errors/tips
  {
    if (compiled.errors && compiled.errors.length) {
      // 打印出错信息：即 compiled.errors 数组的内容
    }
    if (compiled.tips && compiled.tips.length) {
      // 打印提示信息：即 compiled.tips 数组的内容
    }
  }

  // 最终返回这个 res 对象
  var res = {};
  var fnGenErrors = [];

  // 第 2 步，将代码文本转为真正的函数
  /*
    文本（compiled.render）-> 函数（res.render）
    转化过程中发生的错误加入到数组 fnGenErrors 中
   */
  res.render = createFunction(compiled.render, fnGenErrors);

  /*
    文本数组（compiled.staticRenderFns）
    -> 函数数组（res.staticRenderFns）

    转化过程中发生的错误加入到数组 fnGenErrors 中
   */
  res.staticRenderFns = compiled.staticRenderFns.map(
    function (code) {
      return createFunction(code, fnGenErrors)
    }
  );

  {
    if ((!compiled.errors || !compiled.errors.length) 
         && fnGenErrors.length) {
      // 打印文本 -> 函数转化过程中的错误，即数组 fnGenErrors 的内容
    }
  }

  // 将 res 缓存下来，并返回 res
  return (cache[key] = res)
}

// 所以，compileToFunctions 函数返回一个 json：
// 这里的函数都是真正的函数，不像 compiled.render 其实是字符串
{ 
  render: fn , 
  staticRenderFns: [f1, f2, f3, ...]
}
```

看一个官网的例子：

```
// 将字符串模板编译渲染函数
var res = Vue.compile('<div><span>{{ msg }}</span></div>')

// 将渲染函数作为配置对象选项用以创建 Vue 实例
new Vue({
  data: {
    msg: 'hello'
  },
  render: res.render,
  staticRenderFns: res.staticRenderFns
})

```

<h2 id="vue-version">Vue.version</h2>

以字符串形式返回 Vue 的版本号。

```
Vue$3.version = '2.4.0';
```

篇幅有点长，暂时写到这里，后续会继续对 Vue 展开分析。

参考：
[1] https://cn.vuejs.org/v2/api/
[2] http://www.cnblogs.com/jscode/p/3600060.html
















