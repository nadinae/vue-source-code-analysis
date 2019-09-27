# Vuex
每一个vue插件都需要有一个公开的install方法，vuex也不例外。调用了一下applyMixin方法，该方法主要作用就是在所有组件的beforeCreate生命周期注入了设置this.$store这样一个对象

Vue.story会进行一系列的初始化配置，最后会执行`resetStoreVM(this, state)`方法：
```javascript
// src/store.js
function resetStoreVM (store, state, hot) {
  // 省略无关代码
  Vue.config.silent = true
  store._vm = new Vue({
    data: {
      $state: state
    },
    computed
  })
}
```
其本质是将传入的state对象作为一个隐藏的Vue组件的data，也就是说我们commit的时候改变的是data的值，由于data数据的getter和setter是被劫持的。这样就能解释了为什么vuex中的state的对象属性必须提前定义好，如果该state中途增加一个属性，因为该属性没有被defineReactive，所以其依赖系统没有检测到，自然不能更新。
