## 合并配置
Vue的合并配置在执行`_init()`的时候进行
```javascript
Vue.prototype._init = function (options?: Object) {
  // merge options
  if (options && options._isComponent) {
    // optimize internal component instantiation
    // since dynamic options merging is pretty slow, and none of the
    // internal component options needs special treatment.
    initInternalComponent(vm, options)
  } else {
    vm.$options = mergeOptions(
      resolveConstructorOptions(vm.constructor),
      options || {},
      vm
    )
  }
  // ...
}
```
先看`else`的逻辑
```javascript
vm.$options = mergeOptions(
  resolveConstructorOptions(vm.constructor),
  options || {},
  vm
)
```
首先看`resolveConstructorOptions(vm.constructor)`方法传入`vm.constructor`，然后会拿到`Vue.options`，
```javascript
export function resolveConstructorOptions (Ctor: Class<Component>) {
  let options = Ctor.options
  if (Ctor.super) {
    const superOptions = resolveConstructorOptions(Ctor.super)
    const cachedSuperOptions = Ctor.superOptions
    if (superOptions !== cachedSuperOptions) {
      // super option changed,
      // need to resolve new options.
      Ctor.superOptions = superOptions
      // check if there are any late-modified/attached options (#4976)
      const modifiedOptions = resolveModifiedOptions(Ctor)
      // update base extend options
      if (modifiedOptions) {
        extend(Ctor.extendOptions, modifiedOptions)
      }
      options = Ctor.options = mergeOptions(superOptions, Ctor.extendOptions)
      if (options.name) {
        options.components[options.name] = Ctor
      }
    }
  }
  return options
}
```
`Vue.options`这个值是在`initGlobalAPI(Vue)`时定义的，目录`src/core/global-api/index.js`代码：
```javascript
export function initGlobalAPI (Vue: GlobalAPI) {
  // ...
  Vue.options = Object.create(null)
  ASSET_TYPES.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
  })

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue

  extend(Vue.options.components, builtInComponents)
  // ...
}
```
可以看出
  * `Vue.options`最初是通过`Object.create(null)`创建的一个空对象，然后通过遍历`ASSET_TYPES`给`Vue.options`添加新的属性。
  * 将_base设置为Vue对象本身
  * 并扩展了compontents对象
  `ASSET_TYPES`定义在`src/shared/constants.js`，代码：
  ```javascript
  export const SSR_ATTR = 'data-server-rendered'
  export const ASSET_TYPES = [
    'component',
    'directive',
    'filter'
  ]
  export const LIFECYCLE_HOOKS = [
    'beforeCreate',
    'created',
    'beforeMount',
    'mounted',
    'beforeUpdate',
    'updated',
    'beforeDestroy',
    'destroyed',
    'activated',
    'deactivated',
    'errorCaptured',
    'serverPrefetch'
  ]
  ```
  可以看出，在初始化完`Vue.options`后，添加了`component`,`directive`,`filter`三个值为空对象的属性，这个文件还定义了生命周期相关的函数名称。

## mergeOptions方法
定义在`src/core/util/options.js`中：
```javascript
/**
 * Merge two option objects into a new one.
 * Core utility used in both instantiation and inheritance.
 */
export function mergeOptions (
  parent: Object,
  child: Object,
  vm?: Component
): Object {
  if (process.env.NODE_ENV !== 'production') {
    checkComponents(child)
  }

  if (typeof child === 'function') {
    child = child.options
  }

  normalizeProps(child, vm)
  normalizeInject(child, vm)
  normalizeDirectives(child)
  const extendsFrom = child.extends
  if (extendsFrom) {
    parent = mergeOptions(parent, extendsFrom, vm)
  }
  if (child.mixins) {
    for (let i = 0, l = child.mixins.length; i < l; i++) {
      parent = mergeOptions(parent, child.mixins[i], vm)
    }
  }
  const options = {}
  let key
  for (key in parent) {
    mergeField(key)
  }
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key)
    }
  }
  function mergeField (key) {
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
  return options
}
```
接受是哪个参数`parent（Vue的构造函数的options）`，`child（是我们new Vue时传入的数组对象自定义的options）`，`vm（Vue对象本身）`，
首先会对组件的名称进行检测`checkComponents(child)`，
```javascript
function checkComponents (options: Object) {
  for (const key in options.components) {
    validateComponentName(key)
  }
}

export function validateComponentName (name: string) {
  if (!new RegExp(`^[a-zA-Z][\\-\\.0-9_${unicodeRegExp.source}]*$`).test(name)) {
    warn(
      'Invalid component name: "' + name + '". Component names ' +
      'should conform to valid custom element name in html5 specification.'
    )
  }
  if (isBuiltInTag(name) || config.isReservedTag(name)) {
    warn(
      'Do not use built-in or reserved HTML elements as component ' +
      'id: ' + name
    )
  }
}
```
不能使用HTML的保留标签比如`<header></header>`，名称包含字符，数字，连接符，并要以字母开头等。
如果输入的对象是`Function`，那么将`child`赋值为`child.options`
```javascript
normalizeProps(child, vm)
normalizeInject(child, vm)
normalizeDirectives(child)
```
是对`child`的属性进行格式化，因为写法的多样所以会进行一下统一处理


接着会将`extends`和`mixins`合并到`parent`。通过判断是否有`extends`和`mixins`递归调用`mergeOptions`方法。并且`extends`和`mixins`合并到`parent`上面顺序要优先于Vue最初创建的`component`,`directive`,`filter`



## 合并策略
Vue在进行配置合并的时候根据不同的属性执行不同的合并策略，代码`src/core/util/options.js`：
```javascript
const strats = config.optionMergeStrategies
const options = {}
let key
for (key in parent) {
  mergeField(key)
}
for (key in child) {
  if (!hasOwn(parent, key)) {
    mergeField(key)
  }
}
function mergeField (key) {
  const strat = strats[key] || defaultStrat
  options[key] = strat(parent[key], child[key], vm, key)
}
```
`config.optionMergeStrategies`代码是从`src/core/config.js`中引入：
```javascript
  /**
   * Option merge strategies (used in core/util/options)
   */
  // $flow-disable-line
  optionMergeStrategies: Object.create(null),
```
可以看出是创建了一个新对象。
所以`merField()`方法大致就是执行了这样的逻辑
```javascript
var strats = Object.create(null)
strates.data = function () { ... }
strates.created = function () { ... }
strates.watch = function () { ... }
```
这些就是Vue定义的属性特定的合并方法，如果没有的话那么就用默认的合并方法`defaultStrat`。

