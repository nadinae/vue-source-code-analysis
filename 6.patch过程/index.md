## patch过程
createComponent创建组件`VNode`，接着走`vm._update_()`将`VNode`转成真实`DOM`。
`vm._update_()`调用`createElm`创建元素节点，核心代码如下：
```javascript
function createElm (
  vnode,
  insertedVnodeQueue,
  parentElm,
  refElm,
  nested,
  ownerArray,
  index
) {
  // ...
  if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
    return
  }
  // ...
}
```
`createComponent(vnode, insertedVnodeQueue, parentElm, refElm)`判断如果为真直接**return**，看`createComponent()`判断方法：
```javascript
function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {
  let i = vnode.data
  if (isDef(i)) {
    const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
    if (isDef(i = i.hook) && isDef(i = i.init)) {
      i(vnode, false /* hydrating */)
    }
    // after calling the init hook, if the vnode is a child component
    // it should've created a child instance and mounted it. the child
    // component also has set the placeholder vnode's elm.
    // in that case we can just return the element and be done.
    if (isDef(vnode.componentInstance)) {
      initComponent(vnode, insertedVnodeQueue)
      insert(parentElm, vnode.elm, refElm)
      if (isTrue(isReactivated)) {
        reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm)
      }
      return true
    }
  }
}
```
首先判断`vnode.data`是不是一个组件`vnode`,如果是i就是init钩子。在非`keep-alive`情况下调用`createComponentInstanceForVnode`方法返回一个vue实例，并且它会执行实例的`_init`方法，然后调用`$mount`方法挂载子组件。
`createElm`——>
