# 什么是Virtual DOM？
`Virtual DOM`是用js对象来描述真实的`DOM`节点。例如：
```javascript
// 真实的HTML DOM结构
<ul id='list'>
    <li class='item'>Item 1</li>
    <li class='item'>Item 2</li>
</ul>
// 用JS模拟这个DOM
{
  tag: 'ul',
  attrs: {
      id: 'list'
  },
  children: [
      {
        tag: 'li',
        attrs: {className: 'item'},
        children: ['Item 1']
      }, 
      {
        tag: 'li',
        attrs: {className: 'item'},
        children: ['Item 2']
      }
  ]
}
```
用js对象来描述真实的`DOM`节点，一是减少操作真实`DOM`。二是可以用数据驱动`DOM`更新。
打印一个`DOM`节点的内容
```javascript
let oDiv = document.createElement('div');
let str = '';
for(let key in oDiv){
  str += key
}
console.log(str)
```
[!](img.png)