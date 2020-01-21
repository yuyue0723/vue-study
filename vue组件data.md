### 二、Vue组件data选项为什么必须是个函数，而Vue的根实例则没有此限制？
- 1. Vue组件data选项为什么必须是个函数
  + vue组件中data值不能为对象，因为对象是引用类型，组件可能会被多个实例同时引用。如果data值为对象，将导致多个实例共享一个对象，其中一个组件改变data属性值，其它实例也会受到影响。
  + data为函数，通过return返回对象的拷贝，致使每个实例都有自己独立的对象，实例之间可以互相不影响的改变data属性值。

```js
  Vue.component('a-comp', {
    data: function () {
      return {
        foo: 'bar'
      }
    }
  })
```

- 2. 为什么根实例的data是一个对象
  + 因为在Vue中只有一个实例，所以在vue的根实例上可以直接使用对象

  ```js
    new Vue({
      data: {
        foo: 'bar'
      }
    })
  ```
