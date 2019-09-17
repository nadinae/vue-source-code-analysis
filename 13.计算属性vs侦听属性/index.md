# 计算属性vs侦听属性
Vue中计算属性和侦听属性都是调用了vue内部的watcher。

## deep watcher
深度监听
```javascript
var vm = new Vue({
  data() {
    a: {
      b: 1
    }
  },
  watch: {
    a: {
      deep: true,
      handler(newVal) {
        console.log(newVal)
      }
    }
  }
})
vm.a.b = 2
```
此时无法监听到vm.a.b的变化。

# user watcher
普通watch监听方法调用的
侦听属性适用于观测某个值的变化去完成一段复杂的业务逻辑。

# computed watcher
计算属性适合用在模板渲染中，某个值是依赖了其它的响应式对象甚至是计算属性计算而来
