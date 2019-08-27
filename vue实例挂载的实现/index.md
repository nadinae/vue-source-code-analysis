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

在新定义的`$mount`方法中会对挂载的`el`对象进行判断，不允许挂载到`html`或者`body`上。如果挂载到`html`或者`body`上的话在编译模板的时候会覆盖`html`或者`body`标签，导致文档流错误。
在新定义的方法中会进行`if (!options.render)`判断。

+ 如果有`render`方法那么直接调用缓存的`mount`方法。
+ 如果没有`render`方法，在新定义的`$mount`方法中，并且判断有没有写`template`,
  + 有`template`进行判断，拿到模板内容。
  + 没有`template`调用`getOuterHTML`获取挂载对象本身内容

