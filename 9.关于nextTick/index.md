## nextTick

在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

模板
```javascript
<div class="app">
  <div ref="msgDiv">{{msg}}</div>
  <div v-if="msg1">Message got outside $nextTick: {{msg1}}</div>
  <div v-if="msg2">Message got inside $nextTick: {{msg2}}</div>
  <div v-if="msg3">Message got outside $nextTick: {{msg3}}</div>
  <button @click="changeMsg">
    Change the Message
  </button>
</div>
```
Vue 实例
```javascript
new Vue({
  el: '.app',
  data: {
    msg: 'Hello Vue.',
    msg1: '',
    msg2: '',
    msg3: ''
  },
  methods: {
    changeMsg() {
      this.msg = "Hello world."
      this.msg1 = this.$refs.msgDiv.innerHTML
      this.$nextTick(() => {
        this.msg2 = this.$refs.msgDiv.innerHTML
      })
      this.msg3 = this.$refs.msgDiv.innerHTML
    }
  }
})

Message got outside $nextTick: Hello Vue.
Message got inside $nextTick: Hello world.
Message got outside $nextTick: Hello Vue.
```
由此可见当我们用js改变模板中使用的某个`data`数据时，并在`this.msg3 = this.$refs.msgDiv.innerHTML`中立即使用并不会得到更新后的数据。这是因为js是单线程的，并且设置属性值是异步操作当`this.msg3 = this.$refs.msgDiv.innerHTML`获取的时候设置操作并没有完成。所以`Vue`提供了`$nextTick()`方法。`$nextTick()`方法是`DOM`全部渲染完成的回调函数。和`$mount`的区别是`$mount`无法保证子组件的`DOM`渲染完成。


## 什么时候使用Vue.nextTick()

  * 你在`Vue`生命周期的`created()`钩子函数进行的`DOM`操作一定要放在`Vue.nextTick()`的回调函数中。原因是什么呢，原因是在`created()`钩子函数执行的时候DOM 其实并未进行任何渲染，而此时进行`DOM`操作无异于徒劳，所以此处一定要将DOM操作的js代码放进`Vue.nextTick()`的回调函数中。与之对应的就是`mounted`钩子函数，因为该钩子函数执行时所有的DOM挂载和渲染都已完成，此时在该钩子函数中进行任何D`OM`操作都不会有问题 。

  * 在数据变化后要执行的某个操作，当你设置 `vm.someData = 'new value'`，DOM并不会马上更新，而是在异步队列被清除，也就是下一个事件循环开始时执行更新时才会进行必要的DOM更新。如果此时你想要根据更新的 DOM 状态去做某些事情，就会出现问题。为了在数据变化之后等待 `Vue` 完成更新 `DOM` ，可以在数据变化之后立即使用 `Vue.nextTick(callback)` 。这样回调函数在 DOM 更新完成后就会调用。
