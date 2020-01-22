三、你知道vue中key的作用和工作原理吗？说说你对它的理解
+ 1. v-for遍历时，用id,uuid之类作为key，唯一标识节点加速虚拟DOM渲染。
```js
  <ul>
    <li v-for="item in list" :key="item.id">
    </li>
  </ul>
```
  - 如果不使用key，Vue会用一种算法：最小化element的移动，并且会尝试尽最大程度在同适当地地方对相同类型的element，做patch或者reuse。
  - 如果使用了key，Vue会根据keys的顺序记录element，曾经拥有了key的element如果不再出现的话，会被直接remove或者destoryed。

+ 2. 响应式系统没有监听到的数据，用+new Date()生成的时间戳作为key，手动强制触发重新渲染。
```js
  <div :key="rerender">
    <span>Hello Vue.js!</span>
    <complexComponent :propObj="propObj" :propArr="propArr"></complexComponent>
  </div>
  refresh(){
    this.rerender= + new Date()
  }
```
  - 如果使用了key，Vue会根据keys的顺序记录element，曾经拥有了key的element如果不再出现的化，会直接remove或者destoryed。
  - refresh方法调用后，包含了span和complexComponent的div的key发生了变化，也就是说曾经拥有了旧key的div不再出现了，当拥有新值的rerender作为key时，拥有了新key的div出现，那么旧key div会被移除，旧span和complexComponent也会移除，新key div触发渲染，新span，带着父组件新propObj和propArr的新complexComponent渲染。

+ key的原理：
  - vue.js的虚拟DOM算法，在更新vNode时，需要从旧vNode列表中查找与新vNode节点相同的vNode进行更新，如果这个过程设置了属性key，过程就会快很多。
