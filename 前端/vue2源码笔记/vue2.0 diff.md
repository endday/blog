---
title: vue2.0 diff
url: https://www.yuque.com/endday/blog/xv35g7
---

```javascript
function updateChildren(parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
      var oldStartIdx = 0;//旧节点开始index
      var newStartIdx = 0;//新节点开始index
      var oldEndIdx = oldCh.length - 1;//旧节点结束index
      var oldStartVnode = oldCh[0];//旧节点开始节点VNode
      var oldEndVnode = oldCh[oldEndIdx];//旧节点结束节点
      var newEndIdx = newCh.length - 1;//新节点结束index
      var newStartVnode = newCh[0];//新节点开始节点VNode
      var newEndVnode = newCh[newEndIdx];//新节点结束虚拟节点VNode

      //核心dom-diff 算法
      //新旧节点两个指针，做比较
      while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
       //1.旧开始节点 === undefined
        if (isUndef(oldStartVnode)) {
          oldStartVnode = oldCh[++oldStartIdx];
      //2.旧结束节点 === undefined
        } else if (isUndef(oldEndVnode)) {
          oldEndVnode = oldCh[--oldEndIdx];
      //3.旧开始节点 === 新开始节点
        } else if (sameVnode(oldStartVnode, newStartVnode)) {
          patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
          oldStartVnode = oldCh[++oldStartIdx];
          newStartVnode = newCh[++newStartIdx];
          
      //4.旧结束节点 === 新结束节点
        } else if (sameVnode(oldEndVnode, newEndVnode)) {
          patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx);
          oldEndVnode = oldCh[--oldEndIdx];
          newEndVnode = newCh[--newEndIdx];
          
      //5.旧开始节点 === 新结束节点
        } else if (sameVnode(oldStartVnode, newEndVnode)) { 
          patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx);

          //操作真实dom:将老开始节点放置在老结束节点的后面,占了老节点的结束节点位置
          canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm));
          oldStartVnode = oldCh[++oldStartIdx];
          newEndVnode = newCh[--newEndIdx];
          
      //6.旧结束节点 === 新开始节点
        } else if (sameVnode(oldEndVnode, newStartVnode)) { 
          patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
         
          //操作真实dom：将结束节点放置在开始节点前面，因为这里指针有移动，作用就是原本的位置
          canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm);
          oldEndVnode = oldCh[--oldEndIdx];
          newStartVnode = newCh[++newStartIdx];
          
      //7.都不是
        } else {
          if (isUndef(oldKeyToIdx)) { 
            //返回老节点的key-index的映射表
            oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx); 
          }
					// 通过map来查找Vnode、如果没找到也要通过循环查找
          idxInOld = isDef(newStartVnode.key)
            ? oldKeyToIdx[newStartVnode.key]
            : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx);

          //index不存在，就是新增的元素
          if (isUndef(idxInOld)) { 
            //实操dom：新增VNode,并且添加到dom中
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx);
          } else {
            //不是新增元素，则移动
            //需要移动的VNode
            vnodeToMove = oldCh[idxInOld];
            //比较需要移动的VNode和现在新开始节点是否相同
            if (sameVnode(vnodeToMove, newStartVnode)) {
              //打补丁，以及遍历子节点
              patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
             	// 将该下标的值设为undefined，标记着这个Vnode已经被处理过了，避免后面重复处理
              oldCh[idxInOld] = undefined;
           	 	//实操dom
              canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm);
            } else {
              // key相同，但是元素类型不相同，同样是视为需要重新创建节点
              createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx);
            }
          }
          newStartVnode = newCh[++newStartIdx];
        }
      }
      /*
        1.如果开始下标大于结束下标，说明遍历老节点遍历结束
        2.老节点遍历完毕，新节点的下标+1的值，添加进去
        3.如果新节点遍历完了，就删除老节点中开始到结束下标的值
      */
      if (oldStartIdx > oldEndIdx) {
        refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm;
        addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue);
      } else if (newStartIdx > newEndIdx) {
        //老的没有遍历完，新的遍历完了
        //删除老的的节点，从start开始，end结束，包括end
        //这里原先移动了节点，用undefined占位，直接删除不影响任何节点
        removeVnodes(oldCh, oldStartIdx, oldEndIdx);
      }
    }
```

新旧节点数组都配置首尾两个索引
新节点的两个索引：newStartIdx ， newEndIdx
旧节点的两个索引：oldStartIdx，oldEndIdx
以两边向中间包围的形式 来进行遍历
头部的子节点比较完毕，startIdx 就加1
尾部的子节点比较完毕，endIdex 就减1
只要其中一个数组遍历完（startIdx\<endIdx），则结束遍历
![](..\\..\assets\xv35g7\1629027725448-6a30738d-8a79-413a-a1f7-e8dfe01b273a.png)
首先考虑，不移动DOM
其次考虑，移动DOM
最后考虑，新建 / 删除 DOM <a name="LP50B"></a>

### 旧头 == 新头

`patchVnode，newStartIdx ++ ， oldStartIdx ++`
递归更新Vnode，`oldStartIdx `旧头索引加一，`newStartIdx` 新头索引加一
如果不考虑多层DOM 结构，新旧两个头一样的话，就直接进行下一轮循环![](..\\..\assets\xv35g7\1629028544405-4de98814-5189-4fc3-bc5d-de3279b55e84.png) <a name="nY23S"></a>

### 旧尾 == 新尾

和头头相同的处理是一样的，尾尾相同，直接进行下个循环
![](..\\..\assets\xv35g7\1629028561950-12892c34-52c8-4022-a146-fbcfa21d0091.png) <a name="h3YXG"></a>

### 旧头 == 新尾

这种情况为了复用，就要移动dom了
real Dom 真实dom的el现在储存在`oldVode`列表
dom方法中不存在`insertAfter`，所以使用`insertBefore`
即放在 oldEndVnode 后一个节点的前面
然后更新两个索引
`oldStartIdx++，newEndIdx--`

> node.insertBefore(newnode,existingnode)
> existingnode
> 可选。在其之前插入新节点的子节点。如果未规定，则 insertBefore 方法会在结尾插入 newnode。

```javascript
// 源码，应该是为了压缩代码
nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))

// 简化
parentElm.insertBefore(     
  oldStartVnode.elm,      
  oldEndVnode.elm.nextSibling 
);
```

![](..\\..\assets\xv35g7\1629029065947-9429a1b0-e124-47c9-88ea-16ab09a7c827.png) <a name="CYODk"></a>

### 旧尾 == 新头

和上面的情况类似，也要移动dom
![](..\\..\assets\xv35g7\1629029526387-a79d9f6b-c0c5-43c5-b90f-0214cf1cf1b8.png) <a name="ZPM6R"></a>

### 单个遍历查找

前面四种逻辑都不满足的情况
用新子节点的子项，在旧子节点数组中遍历，找出一样的节点
1、生成旧子节点数组以 vnode.key 为key 的 map 表
2、拿到新子节点数组中 一个子项，判断它的key是否在上面的map 中
3、不存在，则新建DOM
4、存在，继续判断是否 sameVnode <a name="H5Wb4"></a>

#### 生成map 表

经过 createKeyToOldIdx 生成一个 map 表 oldKeyToIdx
`{ vnodeKey: 数组Index }`

```javascript
const oldCh = [{    
    tag:"div",  key:1
},{  

    tag:"strong", key:2
},{  

    tag:"span",  key:4
}]

const oldKeyToIdx = {
    1:0,
    2:1,
    4:2
}
```

去匹配map 表，判断是否有相同节点
`oldKeyToIdx[newStartVnode.key]` <a name="UCSWr"></a>

#### key不存在旧子节点数组中

直接创建DOM，并插入oldStartVnode 前面
`createElm(newStartVnode, parentElm, oldStartVnode.elm);`

如果有多个新建节点，但是每次插入都是在`oldStartVnode.elm`前面，顺序如何保证
因为oldStartIdx是递增的，所以这个插入的顺序是正确的
![](..\\..\assets\xv35g7\1629030806165-73d65ca0-ca24-4ff1-a5b6-78d68d8da468.png) <a name="pSARB"></a>

#### key存在旧子节点数组中

找到这个旧子节点，然后判断和新子节点是否 sameVnode
如果相同，直接移动到 oldStartVnode 前面
如果不同，直接创建插入 oldStartVnode 前面 <a name="R3HOs"></a>

### 处理可能剩下的节点

在updateChildren 中，比较完新旧两个数组之后，可能某个数组会剩下部分节点没有被处理过，所以这里需要统一处理 <a name="bAtIU"></a>

#### 新子节点遍历完了

`newStartIdx > newEndIdx`
新子节点遍历完毕，旧子节点可能还有剩
所以我们要对可能剩下的旧节点进行 批量删除。
就是遍历剩下的节点，逐个删除DOM
![](..\\..\assets\xv35g7\1629031334309-39476fea-47a9-4e64-a387-df43a023a724.png) <a name="Fwa7O"></a>

#### 旧子节点遍历完了

`oldStartIdx > oldEndIdx`
旧子节点遍历完毕，新子节点可能有剩
所以要对剩余的新子节点处理
剩余的新子节点不存在旧子节点中，所以全部新建

新建有一个问题，就是插在哪里
`var newEnd = newCh[newEndIdx + 1]`
`refElm = newEnd ? newEnd.elm :null;`
refElm 获取的是 newEndIdx 后一位的节点
当前没有处理的节点是 newEndIdx
也就是说 newEndIdx+1 的节点如果存在的话，肯定被处理过了
![](..\\..\assets\xv35g7\1629032775188-305b3b89-a57c-4ce4-9170-bd065623018c.png) <a name="sJuR5"></a>

### 总结

![](..\\..\assets\xv35g7\1629033010301-a4794d48-d59f-41f8-839f-a1e98ae3204f.png) <a name="YUogv"></a>

#### 头头比较和尾尾比较

![](..\\..\assets\xv35g7\1629033021290-f97cad8d-2587-4fc6-bea0-7dfe0d217972.png) <a name="kZuEn"></a>

#### 旧头新尾比较

![](..\\..\assets\xv35g7\1629033033620-84653bf8-71d9-4a8c-8109-5e646e0ebb61.png) <a name="ViQCy"></a>

#### 旧尾新头比较

![](..\\..\assets\xv35g7\1629033044480-8bc35048-4780-417c-8c1c-d0e6d953da41.png) <a name="qpQu6"></a>

#### 无法复用的节点

![](..\\..\assets\xv35g7\1629033162618-6edaa90b-8578-420f-a5eb-c0ec57ef90d1.png)
newStartIdx> newEndIdx ，结束循环 <a name="pMlhE"></a>

#### 批量删除可能剩下的老节点
