# PerformanceOptimizePratice

### 首屏相关（了解公司对首屏的测量要求，而不是一味地生搬硬套网上的，例如有些测量方式是：一旦用户执行了操作，就认为首屏已经渲染完成）

1.传输相关-包括网络速度和资源包大小
  - 如果是http1，需要严格限制包的大小，把关键的css和立即显示的组件相关的js加载执行，其他组件的样式和组件相关的js可以延迟加载（defineAsyncComponent,etc）
  - 如果是http2，虽然说理论上还是应该遵循这个原则，但不遵循这个原则的时候，（在其他条件一致的情况下）对首屏影响就没这么大
  - 要注意入口的阻塞类的请求，例如那些在mainjs里面，createApp前使用的await请求，如非必要，不要使用，这样会导致，如果A接口花费了500ms，首屏渲染会直接增加500ms
  串行请求也希望能斟酌后使用，这些都会多少影响数据的获取时间
  - 路由懒加载，组件懒加载（这个在http1中会看到明显收益）

2. 尽量减少dom的数量-因为dom的渲染在vue框架下，是直接由js决定的，所以减少dom的渲染约等于减少js的执行时间
- 首屏使用按需加载，懒加载(使用intersectionObsever，usecore/useelementVisible)
- 针对列表和表单这种元素很多的情况，tooltip在初始时使用v-if能非常有效的较少js的执行时间，dom数量，进而减少首屏的时间
- 上面的方法同理适用其他组件，例如点击了才会出现的弹窗，抽屉等，都可以使用v-if让他首屏不要渲染
- select组件里面的options，在非selecting的时候，直接不渲染/销毁，也能非常有效地较少dom的数量
- 冗余的代码及早删掉，冗余的代码会造成多余dom的嵌套，以及不必要的绑定事件

### 交互的流畅程度

- 上面提到的减少dom的数量，在这个环节依然十分有用
- debounce和trottle的使用
- 组件的合理拆分，详见[https://github.com/SukiYuSijing/Interactive-optimization/tree/master/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%E8%BF%87%E7%A8%8B%E8%AE%B0%E5%BD%95](https://github.com/SukiYuSijing/Interactive-optimization/blob/master/%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%E8%BF%87%E7%A8%8B%E8%AE%B0%E5%BD%95/README.md)
- 不合理的正则会导致长任务甚至程序崩溃 
- vue中watch,不需要继续监测时及早destroy.要不然里面watch的callback里可能有长任务，浪费资源
- 识别组件库中有的组件本身存在问题，不使用存在性能问题的方法，例如element-plus的tree组件使用setSelectedNodes的时候会导致超长任务2s+，改成setChecked后解决问题

### 减少网络请求
- 比较稳定的接口，前端直接缓存刷新后第一次请求的数据，一次请求就反复利用 
- 多次请求，抛弃前面的，前面的abort，经常是查询这种场景 
- 多次请求，只响应第一次（正在请求中的那次）请求,或者第一次请求未完成，不允许后面再次发起，适用于表单保存提交等场景

### 从用户心理上减少等待时间

- 骨架屏和loading的使用
- 对于需要等待的数据，渲染空数据的表单->等待接受数据插入数据，比一片空白->出现完整数据，从感官上来看，前者比较快
- 让用户关心的元素及早出现
