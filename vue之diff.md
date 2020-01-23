四、你怎么理解vue中的diff算法？
1. 当数据发生变化时，vue是怎么更新节点的？
渲染真实DOM的开销是很大的，比如有时候我们修改了某个数据，如果直接渲染到真实dom上会引起整个dom树的重绘和重排。使用diff算法，可以只更新我们修改的那一小块dom而不用更新整个dom。
我们先根据真实DOM生成一颗virtual DOM，当virtual DOM某个节点的数据改变后会生成一个新的Vnode，然后Vnode和oldVnode做对比，发现有不一样的地方就直接修改在真实的DOM上，然后使oldVode的值为Vnode。
diff的过程就是调用名为patch的函数，比较新旧节点，一边比较一边给真实的DOM打补丁。
2. virtual DOM和真实DOM的区别
virtual DOM是将真实的DOM的数据抽取出来，以对象的形式模拟树形结构。比如DOM是这样的：
```js
  <div>
    <p>123</p>
  </div>
```
对应的virtual DOM：
```js
var Vnode = {
  tag: 'div',
  children: [
    { tag: 'p', text: '123' }
  ]
}
```
- 注意： VNode和oldNode都是对象，一定要记录

3. diff的比较方式
在采用diff算法比较新旧节点的时候，比较只会在同层级进行，不会跨层级比较。
```js
  <div>
    <p>123</p>
  </div>

  <div>
   <span>456</span>
  </div>
```
上面的代码会分别比较同一层的两个div以及第二层的p和span，但是不会拿div和span作比较。

diff流程
 - 当数据发生改变时，set方法会让调用Dep.notify通知所有订阅者Watcher，订阅者就会调用patch给真实的DOM打补丁，更新相应的视图。

4. vue中的diff算法实现，oldCh：旧的虚拟dom节点数据， newCh：新的虚拟dom节点数据
```js
  function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    var oldStartIdx = 0;
    var newStartIdx = 0;
    var oldEndIdx = oldCh.length - 1;
    var oldStartVnode = oldCh[0];
    var oldEndVnode = oldCh[oldEndIdx];
    var newEndIdx = newCh.length - 1;
    var newStartVnode = newCh[0];
    var newEndVnode = newCh[newEndIdx];
    var oldKeyToIdx, idxInOld, vnodeToMove, refElm;
    // removeOnly is a special flag used only by <transiton-group>
    // to ensure removed elements stay in correct relative positions
    // during leaving transitons
    var canMove = !removeOnly;
    {
      checkDuplicateKeys(newCh)
    }
    // 如果索引正常
    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      // 当前的开始旧节点没有定义，进入下一个节点
      if (isUndef(oldStartVnode)) {
        oldStartVnode = oldCh[++oldStartIdx]; // Vnode has been moved left
        // 当前的结束旧节点没有定义，进入上一个节点
      } else if (isUndef(oldEndVnode)) {
        oldEndVnode = oldCh[--oldEndIdx];
        // 如果旧的开始节点与新的开始节点相同，则开始更新该节点，然后进入下一个节点
      } else if (sameVnode(oldStartVnode, newStartVnode)) {
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue);
        oldStartVnode = oldCh[++oldStartIdx];
        newStartVnode = newCh[++newStartIdx];
        // 如果旧的结束节点与新的开始节点相同，则开始更新该节点，然后进入下一个节点
      } else if (sameVnode(oldEndVnode, newEndVnode)) {
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue);
        oldEndVnode = oldCh[--oldEndIdx];
        newEndVnode = newCh[--newEndIdx];
        // 如果旧的开始节点与新的结束节点相同，更新节点后把旧的开始节点移至节点末尾
      } else if (sameVnode(oldStartVnode, newEndVnode)) {// Vnode moved right
        patchVnode(oldStartVnode, newEndVode, insertedVnodeQueue);
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.eml, nodeOps.nextSibling(oldEndVnode.eml));
        oldStartVnode = oldCh[++oldStartIdx];
        newEndVnode = newCh[--newEndIdx];
        // 如果旧的借宿节点与新的开始节点相同，更新节点后把旧的结束检点移至节点开头
      } else if (sameVnode(oldEndVnode, newStartVnode)) {// Vnode moved left
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue);
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm,oldStartVnode.elm);
        oldEndVnode = oldCh[--oldEndIdx];
        newStartVnode= newCh[++newStartIdx];
      } else {
        // 如果旧的节点没有定义key，则创建key
        if (isUndef(oldKeyToIdx)) { oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx); }
        idxInOld = isDef(newStartVnode.key)
         ? oldKeyToIdx[newStartVnode.key]
         : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx);
        // 如果没有定义index,则创建新的新的节点元素
        if (isUndef(idxInOld)) { // New element
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm);
        } else {
          vnodeToMove = oldCh[idxInOld];
          if (sameVnode(vnodeToMove, newStartVnode)) {
            patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue);
            oldCh[idxInOld] = undefined;
            canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm);
          } else {
            // same key but different element. treat as new element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm);
          }
        }
        newStartVnode = newCh[++newStartIdx];
      }
    }
    // 如果旧节点的开始index大于结束index,则创建新的节点  如果新的开始节点index大于新的结束节点则删除旧的节点
    if ((oldStartIdx > oldEndIdx)) {
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm;
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue);
    } else if (newStartIdx > newEndIdx) {
      removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx);
    }
  }
```

检测newVnode 与 oldVnode是否相同
```js
  function sameVnode(a, b) {
    // 如果key相同、都不是或者是注释节点、都有定义data、都是或者都不是input类型的
    return (
      a.key === b.key && (
        (
          a.tag === b.tag &&
          a.isComment === b.isComment &&
          isDef(a.data) === isDef(b.data) &&
          sameInputType(a, b)
        ) || (
          isTrue(a.isAsyncPlaceholder) &&
          a.asyncFactory === b.asyncFactory &&
          isUndef(b.asyncFactory.error)
        )
      )
    )
  }
  // 如果是input类型，判断是否相同
  function sameInputType (a, b) {
    if (a.tag !== 'input') { return true }
    var i;
    var typeA = isDef(i = a.data) && isDef(i = i.attrs) && i.type;
    var typeB = isDef(i = b.data) && isDef(i = i.attrs) && i.type;
    return typeA === typeB || isTextInputType(typeA) && isTextInputType(typeB)
  }
  // 创建key
  function createKeyToOldIdx (children, beginIdx, endIdx) {
    var i, key;
    var map = {};
    for (i = beginIdx; i <= endIdx; ++i) {
      key = children[i].key;
      if (isDef(key)) { map[key] = i; }
    }
    return map
  }
```

patchVnode更新OldVnode属性与内容(注意：如果鉴定oldVnode与Vnode相等（不是完全相等）只会更新oldVnode的props与text或者增删oldVnode，并不会更新oldVnode的data，所以不建议index作为key)
```js
  function findIdxInOld (node, oldCh, start, end) {
    for (var i = start; i < end; i++) {
      var c = oldCh[i];
      if (isDef(c) && sameVnode(node, c)) { return i }
    }
  }

  function patchVnode (oldVnode, vnode, insertedVnodeQueue, removeOnly) {
    // 如果oldVnode与vnode完全相等直接返回
    if (oldVnode === vnode) {
      return
    }
    var elm = vnode.elm = oldVnode.elm;
    if (isTrue(oldVnode.isAsyncPlaceholder)) {
      if (isDef(vnode.asyncFactory.resolved)) {
        hydrate(oldVnode.elm, vnode, insertedVnodeQueue);
      } else {
        vnode.isAsyncPlaceholder = true;
      }
      return
    }
    // reuse element for static trees.
    // note we only do this if the vnode is cloned -
    // if the new node is not cloned it means the render functions have been
    // reset by the hot-reload-api and we need to do a proper re-render.
    // 如果是静态节点，key一致则直接使用旧节点的组件实例
    if (isTrue(vnode.isStatic) &&
      isTrue(oldVnode.isStatic) &&
      vnode.key === oldVnode.key &&
      (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
    ) {
      vnode.componentInstance = oldVnode.componentInstance;
      return
    }
    var i;
    var data = vnode.data;
    // 更新旧节点的组件实例属性
    if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
      i(oldVnode, vnode);
    }
    var oldCh = oldVnode.children;
    var ch = vnode.children;
    if (isDef(data) && isPatchable(vnode)) {
      for (i = 0; i < cbs.update.length; ++i) { cbs.update[i](oldVnode, vnode); }
      if (isDef(i = data.hook) && isDef(i = i.update)) { i(oldVnode, vnode); }
    }
    // 如果text为空
    if (isUndef(vnode.text)) {
        //新旧子集都有定义
      if (isDef(oldCh) && isDef(ch)) {
          // 如果新的子集不等于旧的子集则更新子集
        if (oldCh !== ch) { updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly); }
      } else if (isDef(ch)) {
          // 如果只有新子集有定义，则添加新的子集
        if (isDef(oldVnode.text)) { nodeOps.setTextContent(elm, ''); }
        addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue);
      } else if (isDef(oldCh)) {
          // 如果只有旧的子集有定义则删除旧的子集
        removeVnodes(elm, oldCh, 0, oldCh.length - 1);
      } else if (isDef(oldVnode.text)) {
          // 如果只有旧子集的text有定义 则把text置空
        nodeOps.setTextContent(elm, '');
      }
      // 如果旧的text不等于新的text则更新为新的text
    } else if (oldVnode.text !== vnode.text) {
      nodeOps.setTextContent(elm, vnode.text);
    }
    // 如果data有定义
    if (isDef(data)) {
      if (isDef(i = data.hook) && isDef(i = i.postpatch)) { i(oldVnode, vnode); }
    }
  }
```
