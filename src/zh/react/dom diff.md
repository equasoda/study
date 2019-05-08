# Dom diff
### 虚拟dom的运行过程

![img](/study/vnode.png)

### 什么是DOM Diff算法?
Web界面由DOM树来构成，当其中某一部分发生变化时，其实就是对应的某个DOM节点发生了变化。在React中，构建UI界面的思路是由当前状态决定界面。前后两个状态就对应两套界面，然后由React来比较两个界面的区别，这就需要对DOM树进行Diff算法分析。

要达到每次界面都可以整体刷新界面的目的，势必需要对算法进行优化。然而Facebook工程师却做到了，他们结合Web界面的特点做出了两个简单的假设，使得Diff算法复杂度直接降低到O(n)
```
1. 两个相同组件产生类似的DOM结构，不同的组件产生不同的DOM结构
2. 对于同一层次的一组子节点，它们可以通过唯一的id进行区分
```
简单来说，diff算法就是当状态发生改变的时候，重新构造一个新的Virtual DOM，然后根据与老的Virtual DOM对比，生成patches补丁，打到对应的需要修改的地方。

### diff 策略

- Web UI中节点跨级操作特别少，可以忽略不计


- 拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构


- 对于同一层级的一组子节点，他们可以通过唯一的key来区分，以方便后续的列表对比算法


基于如上，React分别对tree diff、Component diff 、element diff 进行了算法优化。

#### Tree Diff
概念: 将新旧两颗虚拟 DOM 树,按照层级对应的关系,从头到尾的遍历一遍,,就能找到那些元素是需要更新的,这种方式: Tree Diff

![简析](/study/tree.png)

```
1 只会对相同颜色方框内（同级）的DOM节点进行比较，即同一父节点下的所有子节点
2 当发现节点已经不存在，则该节点及其子节点会被完全删除掉，不会用于进一步的比较
```
![简析](/study/tree1.png)
- 执行过程：create A -> create B -> create C -> delete A

#### Component Diff

不同组件之间的对比 概念: 在对比每一个层级的时候,会有自己的组件,这种组件的对比方式就叫: Component Diff ;

​ 这种对比方法其实比较的就是类型↓↓↓

- 如果类型相同,暂时不更新,

- 如果类型不相同,就需要更新; ( 删除旧的组件,再创建一个新的组件,插入到删除组件的那个位置)

![简析](/study/com.png)

- 执行过程：delete D -> create G

#### Element Diff

同一层级中元素之间的对比
概念: 在类型相同的组件内, 再继续对比组件内部的元素,查看内部元素是否相同,如果需要修改,找到需要修改的元素,进行针对性的修改! 这种方式就叫: Element Diff
```
三种节点操作：
1 INSERT_MARKUP（插入）
2 MOVE_EXISTING（移动）
3 REMOVE_NODE（删除）

INSERT_MARKUP：新的 component 类型不在老集合里，需要对新节点执行插入操作。

MOVE_EXISTING：老的集合包含新的 component 类型，就需要做移动操作，可以复用以前的 DOM 节点。

REMOVE_NODE：老的 component 不在新集合里的，需要执行删除操作 或者 老的 component 类型在新集合里也有，但对应的 element 不同则不能直接复用和更新，需要执行删除操作

```
![简析](/study/ele.png)

![简析](/study/ele1.png)

- 执行过程：B、D 不做任何操作，A、C 进行移动操作

### diff 算法总结:


1. 保持完整的结构,有利于性能的提升


2. 尽量使用相同类型的组件


3. 在进行 Element 或Component 级别对比的时候,为了提高对比的效率, React 推荐我们为每个 for 循环创建出来的元素或者组件,提供一个唯一的 key;


4. Tree diff :将新旧两棵 DOM 树,按照层级对应的关系,这样只需要对树进行一次遍历,就能够找到哪些元素是需要更新的;


5. Component Diff: 在对比每一层的时候,每一层都有自己的组件, 那么组件之间的对比,叫做 Component diff , 对比的方式,就是查看两个组件的类型是否相同,如果相同,则暂时认为不需要更新,如果类型不同,则表示更新(先把旧的组件删除,再创建一个新的组件,插入到刚才删除的位置);


6. Element Diff:如果新旧 DOM 树中的组件类型相同,会继续比较这两个组件内部的元素是否也相同,如果元素发生了改变,则找到需要修改的元素,有针对性的修改,这种组件内部元素级别的对比叫: Element Diff;

#### diff算法实现思路
![简析](/study/line.png)

深入浅出 React [虚拟 DOM Diff 算法解析](https://pc.qq.com/detail/1/detail_2661.html)