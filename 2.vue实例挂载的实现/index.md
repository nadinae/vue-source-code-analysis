# Vue实例挂载的实现
Vue中是通过`$mount`方法去挂载vm的。`$mount`方法在多个文件中都有定义，这些定义和平台、构建方式相关的。这个分析针对`compiler`版本的实现。在这个版本中`$mount`定义在`src/platform/web/entry-runtime-with-compiler.js`文件中。代码如下(截取部分源码)：
```javascript
import Vue from './runtime/index'
import { query } from './util/index'

const mount = Vue.prototype.$mount
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el)
  /* istanbul ignore if */
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }
  const options = this.$options
  // resolve template/el and convert to render function
  if (!options.render) {
    let template = options.template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      template = getOuterHTML(el)
    }
  }
  return mount.call(this, el, hydrating)
}
/**
 * Get outerHTML of elements, taking care
 * of SVG elements in IE as well.
 */
function getOuterHTML (el: Element): string {
  if (el.outerHTML) {
    return el.outerHTML
  } else {
    const container = document.createElement('div')
    container.appendChild(el.cloneNode(true))
    return container.innerHTML
  }
}
Vue.compile = compileToFunctions
export default Vue
```
首先，从`import Vue from ./runtime/index`文件夹下拿到`$mount`方法用`mount`变量将他缓存起来，然后重新在原型上定义`$mount`方法。这样做的意义是为了在` runtime only `版本的Vue中直接用。

在`./runtime/index`文件中的`$mount`方法会接收两个参数，第一个`el`，他表示挂载的元素，可以是字符串也可以是`DOM`对象。第二个参数是和服务端渲染相关，在浏览器环境下我们不需要传第二个参数。

在新定义的`$mount`方法中会对挂载的`el`对象进行判断，不允许挂载到`html`或者`body`上。
```javascript
if (el === document.body || el === document.documentElement) {
  process.env.NODE_ENV !== 'production' && warn(
    `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
  )
  return this
}
```
如果挂载到`html`或者`body`上的话在编译模板的时候会覆盖`html`或者`body`标签，导致文档流错误。
在新定义的方法中会进行`if (!options.render)`判断。

+ 如果有`render`方法那么直接调用缓存的`mount`方法。
+ 如果没有`render`方法，在新定义的`$mount`方法中，并且判断有没有写`template`,
  + 有`template`进行判断，拿到模板内容。
  + 没有`template`调用`getOuterHTML`获取挂载对象本身。赋值给`template`然后根据`template`生成`render`函数
  ```javascript
  if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }
      const { render, staticRenderFns } = compileToFunctions(template, {
        outputSourceRange: process.env.NODE_ENV !== 'production',
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`vue ${this._name} compile`, 'compile', 'compile end')
      }
    }
  ```
所以，Vue不管是写`template`或者是`render`最终都会转成一个`render`函数进行处理。

在`platforms/web/runtime/index`中
```javascript
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```
拿到`el`对象会调用`mountComponent`方法这个方法在`core/instance/lifecycle`文件中定义。在`mountComponent`方法中首先会判断是否有`render`函数，如果没有会创建一个空的`VNode`赋值给`vm.$options.render`代码如下：
```javascript
if (!vm.$options.render) {
  vm.$options.render = createEmptyVNode
  if (process.env.NODE_ENV !== 'production') {
    /* istanbul ignore if */
    if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
      vm.$options.el || el) {
      warn(
        'You are using the runtime-only build of Vue where the template ' +
        'compiler is not available. Either pre-compile the templates into ' +
        'render functions, or use the compiler-included build.',
        vm
      )
    } else {
      warn(
        'Failed to mount component: template or render function not defined.',
        vm
      )
    }
  }
}
```
接着会定义一个`updateComponent`方法，
```javascript
let updateComponent
/* istanbul ignore if */
if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
  updateComponent = () => {
    const name = vm._name
    const id = vm._uid
    const startTag = `vue-perf-start:${id}`
    const endTag = `vue-perf-end:${id}`

    mark(startTag)
    const vnode = vm._render()
    mark(endTag)
    measure(`vue ${name} render`, startTag, endTag)

    mark(startTag)
    vm._update(vnode, hydrating)
    mark(endTag)
    measure(`vue ${name} patch`, startTag, endTag)
  }
} else {
  updateComponent = () => {
    vm._update(vm._render(), hydrating)
  }
}
new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
```
`mountComponent` 核心就是先调用 `vm._render` 方法先生成虚拟 `Node`，再实例化一个渲染`Watcher`，在它的回调函数中会调用 `updateComponent` 方法，最终调用 `vm._update` 更新 `DOM`。
