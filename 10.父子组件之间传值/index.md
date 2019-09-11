## Vue组件之间的通信方式

* Prop(常用)
* $emit(组件封装用的多)
* .sync(语法糖)
* $attrs 和 $listeners (组件封装用的较多)
* provide 和 inject （高阶组件/组件库用的较多）
* 其他方式通信

### Prop(父组件向子组件传值)
```javascript
<div id="div2">
    <input type="text" v-model="parentMsg">
    <component1 v-bind:msg="parentMsg"></component1>
</div>
<script type="text/javascript">
  Vue.component("component1", {
      props: ["msg"],
      template: "<span>{{msg}}</span>"
  })
  new Vue({
      el: "#div2",
      data: {
          parentMsg: "父组件初始化数据"
      }
  })
</script>
```
因为Vue中是单项数据流，父子组件之间的prop形成了单向下行的绑定，父组件的prop更新会流向子组件，但是子组件不会流向父组件。这样是为了防止子组件意外改变父组件的状态，从而导致数据流向难以理解。
有两种情况可能会修改prop的值：
* 子组件接受一个prop，并且希望这个prop当做本地的一个值来使用。这种情况最好定义一个本地的`data`数据。

  ```javascript
  props: ['initialCounter'],
  data: function () {
    return {
      counter: this.initialCounter
    }
  }
  ```
* 这个prop以一个原始的值传入并且需要进行数据转换。此时可以在子组件定义一个计算属性

  ```javascript
  props: ['size'],
  computed: {
    normalizedSize: function () {
      return this.size.trim().toLowerCase()
    }
  }
  ```

### $emit子组件向父组件传递数据

子组件向父组件传递数据是通过`$emit`派发一个事件`vm.$emit('sendMsg','这是子组件派发的数据')`。然后我们可以在父组件中监听这个方法，然后在`methods`中执行操作


```html
<div @sendMsg="getMsg"></div>
```
```javascript
methods:{
  getMsg(val){
    console.log(val) //这是子组件派发的数据
  }
}
```

子组件还可以通过
```javascript
methods:{
  sendFun(){
    vm.$emit('sendMsg','这是子组件派发的数据')
  }
}
```
派发一个方法，在父组件中给子组件的自定义标签加上`ref`属性如`res="child"`，然后就可以在父组件的`methods`方法中调用

```javascript
methods:{
  getMsg(){
    this.$refs.child.sendFun()
  }
}
```

