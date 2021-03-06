---
date: 2020-1-15
complexity: hard
tags: ['react']
---

V16 之前的 React，触发一次渲染，所有操作都是同步的，包括生命周期函数的执行，diff，最终更新 dom 等，假设这个过程会持续 500ms，那么这期间就会一直霸占浏览器渲染进程中的主线程，此时对于用户而言，这个网页是处于卡死状态的，比如在一个 input 框进行键盘输入会得不到即时反应（此类操作是否可以由 Compositor 线程处理？），待 500ms 过后之前输入的一下子蹦出来了。

怎么优化呢？如果把 500ms 的同步任务分片成 50 个 10ms 的任务，虽然总时间没变，但是主线程不会在 500ms 中被一个任务独占，期间可以处理其他优先级更高的任务，之前 input 框键盘输入的情况就会每 10ms 就得到浏览器的反馈，当然上述说的数值都是一个举例。V16 之后的 React 确实是按照这个思路做的，而 Fiber 就是支持时间分片和任务调度的数据结构。

渲染时会创建 WIP 树，挂载在 fiber.alternate 上，而 alternate.alternate 又指向 fiber，形成双向缓存，将要发生的变更都会先作用于 WIP 树，之后再与 fiber 做 diff。

[1]: https://makersden.io/blog/look-inside-fiber
[2]: https://github.com/acdlite/react-fiber-architecture
[3]: https://zhuanlan.zhihu.com/p/26027085
[4]: https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3DZCuYPiUIONs
