# Vue中的render方法
当执行Vue脚手架命令的时候会提示安装版本
<pre>
Runtime + Compiler: recommended for most users
Runtime-only: about 6KB lighter min+gzip, but templates (or any Vue-specific HTML) are ONLY allowed in .vue files - render functions are required elsewhere
</pre>
`Runtime-only`版本就会生成`render: h => h(App)`这样的一个函数。
`Runtime + Compiler`版本是需要传入`template`模板字符串，然后最终会将`template`模板字符串编译成模板字符串。

Vue的`_render`方法是定义在原型上的私有方法，它用来把实例渲染成一个虚拟Node，`_render`方法定义在`src/core/instance/render.js`文件中，截取部分源码如下：
```javascript
Vue.prototype._render = function (): VNode {
  const vm: Component = this
  const { render, _parentVnode } = vm.$options

  if (_parentVnode) {
    vm.$scopedSlots = normalizeScopedSlots(
      _parentVnode.data.scopedSlots,
      vm.$slots,
      vm.$scopedSlots
    )
  }

  // set parent vnode. this allows render functions to have access
  // to the data on the placeholder node.
  vm.$vnode = _parentVnode
  // render self
  let vnode
  try {
    // There's no need to maintain a stack because all render fns are called
    // separately from one another. Nested component's render fns are called
    // when parent component is patched.
    currentRenderingInstance = vm
    vnode = render.call(vm._renderProxy, vm.$createElement)
  } catch (e) {
    ......
  }
  // set parent
  vnode.parent = _parentVnode
  return vnode
}
```
可以看到从`vm.$options`中拿到`render`对象`const { render, _parentVnode } = vm.$options`然后定义一个`vnode`变量，然后将`render`方法绑定到`vm._renderProxy`传入`vm.$createElement`参数，并赋值给`vnode`。`vnode = render.call(vm._renderProxy, vm.$createElement)`

# vm._renderProxy
`vm._renderProxy`定义在`src/core/instance/init.js`中。
```javascript
if (process.env.NODE_ENV !== 'production') {
  initProxy(vm)
} else {
  vm._renderProxy = vm
}
```
在生产环境中直接将`vm._renderProxy`赋值为Vue实例。在开发环境中会调用`initProxy(vm)`方法，定义在`src/core/instance/proxy.js`中。源码如下：
```javascript
initProxy = function initProxy (vm) {
  if (hasProxy) {
    // determine which proxy handler to use
    const options = vm.$options
    const handlers = options.render && options.render._withStripped
      ? getHandler
      : hasHandler
    vm._renderProxy = new Proxy(vm, handlers)
  } else {
    vm._renderProxy = vm
  }
}
```
首先判断浏览器是否支持es6的`hasProxy`方法，设置拦截器`handlers`为`getHandler`或者`hasHandler`。`options.render._withStripped`属性只在测试代码中被设置为true。

# 关于hasHandler和getHandler

`hasHandler`代码如下：
```javascript
const hasHandler = {
  has (target, key) {
    const has = key in target
    const isAllowed = allowedGlobals(key) ||
      (typeof key === 'string' && key.charAt(0) === '_' && !(key in target.$data))
    if (!has && !isAllowed) {
      if (key in target.$data) warnReservedPrefix(target, key)
      else warnNonPresent(target, key)
    }
    return has || !isAllowed
  }
}

const getHandler = {
  get (target, key) {
    if (typeof key === 'string' && !(key in target)) {
      if (key in target.$data) warnReservedPrefix(target, key)
      else warnNonPresent(target, key)
    }
    return target[key]
  }
}
```
首先会进行两个判断：
* 判断key属性是否存在在target中`const has = key in target`
* 是全局变量或者是以_开头的字符串并且没有在$data中，返回true
  +如果在$data中也没有找到key就会调用`warnNonPresent`方法。
  +如果有的话触发`warnReservedPrefix`方法。

# 关于warnNonPresent和warnReservedPrefix
```javascript
const warnNonPresent = (target, key) => {
  warn(
    `Property or method "${key}" is not defined on the instance but ` +
    'referenced during render. Make sure that this property is reactive, ' +
    'either in the data option, or for class-based components, by ' +
    'initializing the property. ' +
    'See: https://vuejs.org/v2/guide/reactivity.html#Declaring-Reactive-Properties.',
    target
  )
}

const warnReservedPrefix = (target, key) => {
  warn(
    `Property "${key}" must be accessed with "$data.${key}" because ` +
    'properties starting with "$" or "_" are not proxied in the Vue instance to ' +
    'prevent conflicts with Vue internals. ' +
    'See: https://vuejs.org/v2/api/#data',
    target
  )
}
```
## warnReservedPrefix
当我们访问Vue data里面属性的时候可以直接`vm.xxx`访问，而不用`vm.$data.xxx`是因为Vue对data里面的属性做了一层代理，但是当我们data里面的属性值以 **_** 开头时，为了不和Vue的私有属性冲突就不会做代理，可以通过`vm.$data._xxx`访问，如果设置了以 **_** 开头的属性并且用`vm.xxx`访问，就会触发`warnReservedPrefix`方法，例如：
```javascript
var vm = new Vue({data: {_name: 1}})
vm._name // undefined
vm.$data._name // 1
```
## warnNonPresent
如果访问一个没有定义的属性的时候就会触发`warnNonPresent`方法。

# render方法的参数vm.$createElement
render方法的参数`vm.$createElement`就是在我们实例化Vue的时候传入的render函数,
```javascript
var vm = new Vue({
  data: {
    _name: 1
  },
  render: function (createElement) {
    return createElement('div', {
      attrs: {
          id: 'app'
        },
    }, this.message)
  }
})
```
`vm.$createElement`方法在`render.js`文件有定义
```javascript
// bind the createElement fn to this instance
// so that we get proper render context inside it.
// args order: tag, data, children, normalizationType, alwaysNormalize
// internal version is used by render functions compiled from templates
vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
// normalization is always applied for the public version, used in
// user-written render functions.
vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
```
根据注释可以判断`vm._c`就是`template`被编译成`render`函数使用。
`vm.$createElement`是用户手写`render`方法使用的。

**最终**`createElement`**会返回创建好的VNode。然后会执行_update方法渲染成真是DOM，而_update方法会在首次渲染和数据更新的时候触发**
