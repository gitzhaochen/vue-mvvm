## 手写Vue双向绑定

1. 初始化监听data对象，Object.defineProperty(data) get方法同时会收集当前属性有多少个订阅者，当更新的时候通知这些订阅者
2. 编译模版、填充data值
    - 如果节点类型是input，添加input\keyup\change等事件监听、回调触发set事件
    - 如果节点类型是文本，添加订阅者watcher，当数据变化同步更新nodeValue
3. input输入变化 -- 触发Object.defineProperty set() -- set发出更新通知dep.notify() -- 订阅者收到通知并更新 watcher.update()

- [数据初始化绑定，编译模版](./bind-compile.html)
- [响应式的数据绑定，输入框变化同步更新data值](./view-to-model.html)
- [订阅/发布模式，data更新同步视图](./publiser-watcher.html)