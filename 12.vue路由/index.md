# 路由模式
vue-router提供三种路由模式：
* hash:使用URL  hash值来做路由。默认模式
* history:利用HTML5 history API和服务器配置。
* abstract:支持所有的JavaScript运行环境，如Node.js服务端

## hash模式
`hash`即浏览器 **#** 后面的部分，包括 **#**。`hash`是URL中的锚点，代表的是网页中的一个位置，只是改变 **#** 后面的部分，从而触发 **hashChange**事件，从而加载不同的内容，`hash`的改变不会重新请求服务器，从而不会导致页面的重新加载。

* `hash`的改变不会请求服务器，利用这一特点监听`hashChange`事件
* 每一次改变`hash`就会往浏览器的访问历史中增加一个记录，可以使用”后退”按钮，就可以回到上一个位置。

## history模式
HTML5 History API提供了一种功能，能让开发人员在不刷新整个页面的情况下修改站点的URL，就是利用 history.pushState API 来完成 URL 跳转而无须重新加载页面；`history`模式不会带`hash`模式的 **#** 。


## hash模式和history模式的区别
由于history模式是改变的真是的路由地址，在我们刷新的时候会发送真是的http请求，如果后端没有配置对应的path，那么将会出现 **404**。

## abstract模式
