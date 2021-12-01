

# Home
[Home](https://github.com/yangzhe327/Front-end-Study-Notes)

## 生命周期
作者：WashingtonHua
链接：https://juejin.cn/post/6844903834200866829
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 1.1 总结
1.同步路由，父组件在 render 阶段创建子组件。
2.异步路由，父组件在自身挂载完成之后才开始创建子组件。
3.挂载完成之后，在更新时，同步组件和异步组件是一样的。
4.无论是挂载还是更新，以 render 完成为界，之前父组件先执行，之后子组件先执行。
5.兄弟组件大体上按照在父组件中的出场顺序执行。
6.useEffect 会在挂载/更新完成之后，延迟执行。
7.异步请求（如访问 API）何时得到响应与组件的生命周期无关，即父组件中发起的异步请求不保证在子组件挂载完成前得到响应。

### 1.2 挂载过程
父子组件的挂载分为三个阶段。
第一阶段，父组件执行到自身的 render，解析其下有哪些子组件需要渲染，并对其中同步的子组件进行创建，挨个执行各组件到 render，生成到目前为止的 Virtual DOM 树，并 commit 到 DOM。
第二阶段，此时 DOM 节点已经生成完毕，组件挂载完成，开始后续流程。先依次触发同步子组件各自的 componentDidMount / useLayoutEffect，最后触发父组件的。
第三阶段，如果组件使用了 useEffect，则会在第二阶段之后触发 useEffect。如果父子组件都使用了 useEffect，那么**子组件先触发，然后是父组件。**
如果父组件中包含异步子组件，则会在父组件挂载完成后被创建。
对于兄弟组件，如果是同步路由，它们的创建顺序和在父组件中定义的出场顺序是一致的。

对于「异步的兄弟组件」，最终的加载顺序是按照 JSX 中定义的顺序，还是按照 js 文件下载完成的顺序，我暂时还不能确定。
按照我对“异步”的理解，我更倾向于认为是按照下载完成的顺序，这更符合“按需加载”的概念。
之所以会造成困扰，是因为据我目前所观察到的情况，两种顺序是一致的，我还没有遇到过后定义但先加载的情况。
大部分时候我们会以页面为单位去划分异步组件，单个页面需要加载多个异步组件的场景比较少；即便在这些少数场景中，单次需要请求的文件数量也不会很多，不至于超过浏览器的并发上限；即便超过，也会按照在父组件中定义的出场顺序去分批发起请求。考虑到单个异步组件的文件尺寸通常都很小，加载速度非常快，同一批发起的请求基本上也都是同时到达，因此大部分时候下载完成的顺序和定义的顺序是一致的。
但没遇到不代表不存在，该问题我会进一步验证，已经有结果的小伙伴也可以分享一下。

如果组件的初始化过程包含异步操作（通常在 componentDidMount() 和 useEffect(fn, []) 中进行），这些操作何时得到响应与组件的生命周期无关，完全看异步操作本身花了多少时间。

### 1.3 更新过程
React 的设计遵循单向数据流模型，兄弟节点之间的通信也会经过父组件（Redux 和 Context 也是通过改变父组件传递下来的 props 实现的），因此任何两个组件之间的通信，本质上都可以归结为父组件更新导致子组件更新的情况。
父子组件的更新同样分为三个阶段。
第一、三阶段，和挂载过程基本一样，无非是第一阶段多了一个 Reconciliation 的过程，第三阶段需要先执行 useEffect 的 Cleanup 函数。
第二阶段，和挂载过程也很类似，都是子组件先于父组件，但更新比挂载涉及的函数要多一些：

getSnapshotBeforeUpdate()
useLayoutEffect() 的 Cleanup
useLayoutEffect() / componentDidUpdate()

React 会按照上面的顺序依次执行这些函数，每个函数都是各个子组件的先执行，然后才是父组件的执行。具体说来，就是先执行各个子组件的 getSnapshotBeforeUpdate()，然后是父组件的 getSnapshotBeforeUpdate()，再然后是各个子组件的 componentDidUpdate()，父组件的 componentDidUpdate()，以此类推。
这里我们把类组件和 Hooks 的生命周期函数放在了一起，因为父子组件可以是这两种组件类型的任意排列组合。实际渲染时不一定每一个函数都有用到，只会调用组件实际拥有的函数。

### 1.4 卸载过程
卸载过程涉及到 componentWillUnmount()、useEffect() 的 Cleanup、useLayoutEffect() 的 Cleanup 这三种函数，顺序固定为父组件的先执行，子组件按照在 JSX 中定义的顺序依次执行各自的方法。
注意，此时的 Cleanup 函数会按照在代码中定义的顺序先后执行，与函数本身的特性无关。
如果卸载旧组件的同时伴随有新组件的创建，新组件会先被创建并执行完 render，然后卸载不需要的旧组件，最后新组件执行挂载完成的回调。


## useState
  ### 1.setState为异步调用
  ### 2.useState()的参数是个函数
    useState的初始值如果是个执行函数，每次render的时候都会重新调用这个执行函数，但并不进行赋值
    ```
    const [state, setState] = useState(func())
    ```
    声明函数只会在第一次render的时候调用并且赋值
    ```
    const [state, setState] = useState(() => func())
    ```

## useEffect
  ### 1.useEffect为异步调用
  ### 2.如果父子组件都使用了 useEffect，那么**子组件先触发，然后是父组件。**
  
## useLayoutEffect
  根据 React 的官方文档，useEffect() 和 useLayoutEffect() 都是等效于 componentDidUpdate() / componentDidMount() 的存在，但实际上两者在一些细节上还是有所不同：
  ### 1.useLayoutEffect为同步调用，而useEffect为异步调用
  useLayoutEff() 永远比 useEffect() 先执行，即便在你的代码中 useEffect() 是写在前面的。所以 useLayoutEffect() 才是事实上和 componentDidUpdate() / componentDidMount() 平起平坐的存在。useEffect() 会在父子组件的 componentDidUpdate() / componentDidMount() 都触发之后才被触发。当父子组件都用到 useEffect() 时，子组件中的会比父组件中的先触发。
  ### 2.不团结的Cleanup
  同样都拥有 Cleanup 函数，useLayoutEffect() 和它的 Cleanup 未必是挨着的。
当父组件是 Hooks、子组件是 Class 时，能够很明显看出，useLayoutEffect() 的 Cleanup 会在 getSnapshotBeforeUpdate() 和 componentDidUpdate() 之间被调用，而 useLayoutEffect() 则是和 componentDidUpdate() 同级，按照更新过程的顺序被调用。
Hooks 作为子组件时也是这么个过程，只是没有了子组件，看上去不那么明显罢了。
而 useEffect() 就不一样，它和它的 Cleanup 紧密团结在一起，每次执行都是前后脚一起的，从不分离。
  
  
