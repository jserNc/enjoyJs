---
title: Vue 实例创建过程
date: 2017-12-21 09:09:26
tags: vue
---

基于 Vue 的应用往往开始于新建一个 Vue 实例，将一个 options 配置对象传给 Vue 构造函数即可完成实例的创建，本文以一个小例子来梳理一下 Vue 实例的创建过程。

<!-- more -->

下面是官网上的第一个例子，梳理一下这段代码的执行流程。

```
// dom 模板
<div id="app">
  {{ message }}
</div>

// 创建 Vue 实例
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

首先，执行构造函数 Vue（[上篇已经分析过，这里的 Vue$3 就是我们用到的 Vue](http://nanchao.win/2017/12/19/vue-global-api/))

``` 
function Vue$3 (options) {
  /* 若直接调用 Vue 打印警告，Vue 是构造函数，必须用 new 来调用 */

  // 根据 options 完成一系列初始化操作
  this._init(options);
}

// 于是，以上代码转化为：
var vm = {};
vm.__proto__ = Vue$3.prototype;
vm._init({
  del: '#app',
  data: {
    message: 'Hello Vue!'
  }
});
```

然后，执行初始化函数 Vue.prototype._init

```
Vue.prototype._init = function (options) {
  var vm = this;
  // uid$1 是全局数值变量，每个 Vue 实例有一个唯一的 _uid
  vm._uid = uid$1++;

  // 非生产模式下记录初始化开始时间

  // 标志当前对象是 Vue 实例
  vm._isVue = true;

  // ① options 用来创建组件
  if (options && options._isComponent) {
    // 用 options 对 vm.$options 进行修正
    initInternalComponent(vm, options);
  // ② options 用来创建 vm 实例
  } else {
    // vm.$options = vm.constructor.options + options
    vm.$options = mergeOptions(
      // 返回子类构造函数的 options，即 vm.constructor.options
      // 子类 Ctor 选项 = 父类 Ctor 选项 + 子类 Ctor 扩展选项
      resolveConstructorOptions(vm.constructor),
      options || {},
      vm
    );
  }

  // 初始化代理，即添加 vm._renderProxy 属性
  initProxy(vm);
  
  // 保留对自身的引用
  vm._self = vm;

  /*
    初始化声明周期，即添加下列属性：
    vm.$parent = parent;     // 最近的非抽象父元素
    vm.$root = parent ? parent.$root : vm; // 根 Vue 实例
    vm.$children = [];       // 子组件
    vm.$refs = {};           // 元素/组件的引用
    vm._watcher = null;//new Watcher(vm,updateComponent,noop)
    vm._inactive = null;     // 布尔值，失效状态
    vm._directInactive = false; // 布尔值，失效状态
    vm._isMounted = false;   // 标志是否插入过文档
    vm._isDestroyed = false; // 标志是否被销毁
    vm._isBeingDestroyed = false; // 标志是否正在被销毁
   */
  initLifecycle(vm);

  /*
    初始化事件，即添加下列属性：
    vm._events = Object.create(null); // “事件类型-回调”映射表
    vm._hasHookEvent = false;         // 是否拥有钩子事件

    然后，若父级事件存在，就添加到 vm._events 中
   */
  initEvents(vm);

  /*
    初始化渲染，即添加下列属性：
    vm._vnode = null;  // 子树的根
    vm._staticTrees = null; // 静态树组成的数组
    vm.$vnode = vm.$options._parentVnode;  // 父树中的占位节点
    vm.$slots = resolveSlots(vm.$options._renderChildren, 
                             renderContext);
    vm.$scopedSlots = emptyObject; // 作用域插槽
    
    // 创建 vnode（内部版本，最后参数始终为 false）
    vm._c = function (a, b, c, d) { 
      return createElement(vm, a, b, c, d, false); 
    };
    
    // 创建 vnode（公开版本，最后参数始终为 true）
    vm.$createElement = function (a, b, c, d) { 
      return createElement(vm, a, b, c, d, true); 
    };

    // vm.$attrs = vm.$options._parentVnode.data.attrs;
    // vm.$listeners = vm.$options._parentVnode.data.on;
   */
  initRender(vm);


  // 调用 beforeCreate 钩子回调函数
  callHook(vm, 'beforeCreate');

  // 初始化 inject（在初始化 data/props 之前）
  initInjections(vm);

  // 初始化 props、methods、data、computed、watch 等
  initState(vm);

  // 初始化 provide（在初始化 data/props 之前）
  initProvide(vm);

  // 调用 created 钩子回调函数
  callHook(vm, 'created');

  // 非生产模式下记录初始化结束时间

  // vm 挂载到元素 el 上
  if (vm.$options.el) {
    vm.$mount(vm.$options.el);
  }
};
```

可以看到，vm._init() 的作用是进行一系列的初始化操作，下面重点看下 vm.$mount(vm.$options.el)：

```
Vue$3.prototype.$mount = function (el,hydrating) {
  // 挂载的元素
  el = el && inBrowser ? query(el) : undefined;
  // 新建一个 Watcher 实例观察 vm 的变动
  return mountComponent(this, el, hydrating)
};
```

其实，这不是真正的 Vue$3.prototype.$mount 方法，后面还有：

```
// 保存之前定义的 Vue$3.prototype.$mount
var mount = Vue$3.prototype.$mount;
// 重新定义 Vue$3.prototype.$mount
Vue$3.prototype.$mount = function (el,hydrating) {
  el = el && query(el);

  if (el===document.body || el===document.documentElement){
    /* 警告：不能将挂载到 <html> 或 <body>，只能挂载到普通元素上 */
    return this
  }

  var options = this.$options;
 
  // 将 template/el 转成 options.render
  if (!options.render) {

    var template = options.template;

    // (1) template 存在，有几种形式
    if (template) {
      // ① template 为字符串形式
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          // 根据选择器 id 获取元素，然后返回该元素的 innerHTML
          template = idToTemplate(template);
          // 如果没找到对应元素，发出警告
        }
      // ② template 为 dom 节点
      } else if (template.nodeType) {
        template = template.innerHTML;
      // ③ 其他都是无效的 template 选项
      } else {
        /* 警告：无用 template */
        return this
      }
    // (2) template 不存在，el 也可当做 template
    } else if (el) {
      template = getOuterHTML(el);
    }

    // 根据模板 template 生成渲染函数
    if (template) {
      /* 标记编译开始 */
    
      /*
       compileToFunctions (template, options, vm) 生成渲染函数
       结果为：
       {
          render : f,
          staticRenderFns : [f,f,f,...]
       }
      */
      var ref = compileToFunctions(template, {
        shouldDecodeNewlines: shouldDecodeNewlines,
        delimiters: options.delimiters,
        comments: options.comments
      }, this);
      
      var render = ref.render;
      var staticRenderFns = ref.staticRenderFns;

      // 修改 options 对象中的渲染函数
      options.render = render;
      options.staticRenderFns = staticRenderFns;

      /* 标记编译结束 */
    }
  }
  // 修正完 this.$options 的渲染函数，开始挂载
  return mount.call(this, el, hydrating)
};
```

继续往下，得回到 mountComponent(this, el, hydrating)：

```
function mountComponent (vm, el, hydrating) {
  vm.$el = el;

  // 没有自定义 render 方法，发出警告
  if (!vm.$options.render) {
    // 修正 render 方法，作用为创建空元素（注释）
    vm.$options.render = createEmptyVNode;
    {
      // ① 模板需要从 dom 元素编译而成，发出警告
      if ((vm.$options.template 
           && vm.$options.template.charAt(0) !== '#') 
           || vm.$options.el || el) {
        /*
          ① template 为 dom 元素 id
          ② el 存在

          两种情况本质都是模板取自 dom 元素，那就发出警告
          （因为当前版本模板编译器不可用）
         */
      // ② 直接发出警告，模板或渲染函数不存在
      } else {
          /* 警告，组件安装失败：未定义模板或者渲染函数 */
      }
    }
  }

  // 调用 beforeMount 钩子回调
  callHook(vm, 'beforeMount');

  var updateComponent;
  
  // 视图更新函数
  updateComponent = function () {
    // 生成新的虚拟 dom 树
    var vnode = vm._render();
    // 根据虚拟 dom 树来更新视图
    vm._update(vnode, hydrating);
  };

  // 创建观察者实例（也正是这句代码触发 'Hello Vue!' 显示在页面）
  vm._watcher = new Watcher(vm, updateComponent, noop);

  hydrating = false;

  // $vnode 为 vm 在父树种的占位节点
  // 现在占位节点为 null，就是挂载成功了
  if (vm.$vnode == null) {
    // 标记挂载成功
    vm._isMounted = true;
    // 调用 mounted 钩子回调
    callHook(vm, 'mounted');
  }
  return vm
}
```

到这里我们看到，要想 vm 对象的 data 数据（'Hello Vue!'）在页面上显示出来，就得调用 updateComponent 函数，其实这个函数就是在创建 vm._watcher 这个 Watcher 实例的时候执行的。这里简要说明一下原因：

```
// Watcher 原型 get 方法
Watcher.prototype.get = function get () {
  ...
  try {
    // this.getter 执行时的 this 和实参都为 vm
    value = this.getter.call(vm, vm);
  } 
  ...
  return value
};

// Watcher 构造函数
var Watcher = function Watcher (vm, expOrFn, cb, options) {
  ...
  if (typeof expOrFn === 'function') {
    this.getter = expOrFn;
  }

  /*
    vm._watcher = new Watcher(vm, updateComponent, noop)
    这句代码会执行 updateComponent 函数，原因如下：

    分解上面的 new 运算符执行过程：
    vm._watcher = {}；
    vm._watcher._proto__ = Vue$3.prototype;
    ...
    vm._watcher.value = vm._watcher.get();

    vm._watcher.get() 会触发 vm._watcher.getter()
    也就是 expOrFn()，即 updateComponent()
   */
  this.value = this.lazy ? undefined : this.get();
};
```

到这里，实例的创建展示流程粗略地走了一遍，细节问题后续再分析。



参考：
[1] https://cn.vuejs.org/v2/guide/instance.html
[2] https://cn.vuejs.org/v2/api/