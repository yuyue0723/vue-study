### 一、v-if和v-for哪个优先级高？如果两个同时出现，应该怎么优化得到更好的性能？
- 优先级
  + v-for与v-if一起使用时，v-for比v-if具有更高的优先级，这意味着v-if将分别重复运行于每个v-for循环中，因此不建议同时使用v-for与v-if。
- 如何同时使用
  + 1. 将 v-if 和 v-for 分别放在不同标签中
  ```js
    <ul v-if="active">
      <li v-for="item in users" :key="item.id">
        {{item.name}}
      </li>
    </ul>
  ```
  + 2. 如果 v-if 和 v-for 只能放在同一级标签中，使用计算属性
  ```js
    <ul>
      <li v-for="item in activeUsers" :key="item.id">
        {{item.name}}
      </li>
    </ul>

    computed: {
      activeUsers: function () {
        return this.users.filter(function(item){
          return item.active
        })
      }
    }
  ```
