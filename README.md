
# Promises/A\+



> **这是一个开放标准，旨在让不同开发者实现的 JavaScript Promise 能够无缝衔接并应用——由前辈们制定，为其他后来者提供参考**


一个 *promise* 所表示的是异步操作的结果。与 *promise* 交互的主要方式是通过它的 `then` 方法，该方法会注册所传入的回调函数，回调函数将接收 *promise* 的最终值或者 *promise* 无法被满足时的原因。


本规范详细说明了 `then` 方法的行为，为所有符合 Promises/A\+ 标准的 *promise* 实现提供可靠的基础。因此本规范的更新或修订应被视为非常稳定。虽然 Promises/A\+ 组织可能会偶尔对本规范进行向后兼容的小幅度修订，以解决新发现的边缘情况，但我们依然会在经过认真仔细的考虑、讨论和测试后，才会去整合重大或者不向后兼容的变更。


在过去，Promises/A\+ 不仅明确早期 [Promise/A](https://github.com) 提案中的涉及的行为条款，而且还扩展了一些涵盖实际行为的内容，并且省略了那些不清不楚或存在问题的部分。


最后要说的是，Promises/A\+ 规范的核心并不涉及如何创建、满足或拒绝 *promise*，而是选择专注于提供一个强大的 `then` 方法以确保不同实现之间的兼容性。大概未来的相关规范才可能会涉及如何创建、满足和拒绝 *promise* 这些主题吧。


## 1 专业术语


1. `thenable`： 一个具有 `then` 方法的对象或函数。
2. `promise`：一个具有 `then` 方法的对象或函数，`then` 方法的行为符合本规范。
3. `value`：是一个合法的 JS 值，含 `undefined`、`null`、`thenable` 或 `promise` 等。
4. `exception`：通过 `throw` 语句抛出的值。
5. `reason`：通常用于表示 `promise` 被拒绝/无法实现的原因，也是个值。


## 2 详细规范


### 2\.1 Promise 状态


一个 promise 必须处于以下三种状态之一：


`待定中（pending）`、`已实现（fulfilled）`、`已拒绝（rejected）`


关于这些状态的描述如下：


1. 待定中 —— pending:
	1. 可以转变为其它两种状态，也就是已实现（fulfilled）或已拒绝（rejected）
2. 已实现 —— fulfilled:
	1. 禁止从该状态转变为其它状态，状态不可改变
	2. 必须有一个不可改变的值（value —— 完成后的成果！！）
3. 已拒绝 —— rejected:
	1. 禁止从该状态转变为其它状态，状态不可改变
	2. 必须有一个不可改变的原因（reason —— 为什么拒绝！？）



> 注意：`不可改变`并不意味着深层次的不可变（比如属性描述符中 `writeable: false`），你可以通过对当前状态进行全等判断（`===`），从而更改相应的值/原因和状态后，那么`不可改变`自然而然的就会成立了。


### 2\.2 `then` 方法


一个 *promise* 必须提供一个 `then` 方法，它可以访问当前或最终的值或者原因。


*promise* 的 `then` 方法接受两个参数：



```
promise.then(onFulfilled, onRejected)

```

1. `onFulfilled` 和 `onRejected` 均是可选参数：


	1. 如果 `onFulfilled` 不是一个函数，则必须被忽略。
	2. 如果 `onRejected` 不是一个函数，则必须被忽略。
> `被忽略`是指我们不对它做任何的处理，不会抛出任何的错误
2. 如果 `onFulfilled` 是一个函数：


	1. 必须在 `promise` 被实现后调用，且以 `promise` 的值（value）作为第一个参数。
	2. 在 `promise` 被实现之前不可调用。
	3. 不允许被多次调用。
3. 如果 `onRejected` 是一个函数：


	1. 必须在 `promise` 被拒绝后调用，且以 `promise` 的原因（reason）作为第一个参数。
	2. 在 `promise` 被拒绝之前不可调用。
	3. 不允许被多次调用。
4. `onFulfilled` 或 `onRejected` 必须在执行上下文栈仅包含平台代码时才被调用。\[[3\.1](https://github.com)]
5. `onFulfilled` 和 `onRejected` 必须作为函数调用（没有 `this` 值）。\[[3\.2](https://github.com)]
6. `then` 可以在同一个 promise 上被多次调用。


	1. 当 `promise` 被实现时，所有相应的 `onFulfilled` 回调必须按它们调用 `then` 的顺序执行。
	2. 当 `promise` 被拒绝时，所有相应的 `onRejected` 回调必须按它们调用 `then` 的顺序执行。
7. `then` 必须返回一个 promise \[[3\.3](https://github.com)]。



```
promise2 = promise1.then(onFulfilled, onRejected);

```

	1. 如果 `onFulfilled` 或 `onRejected` 返回一个值 `x`，则运行 Promise 解决过程 `[[Resolve]](promise2, x)`\[[2\.3](https://github.com)]。
	2. 如果 `onFulfilled` 或 `onRejected` 抛出异常 `e`，则 `promise2` 必须以 `e` 作为原因被拒绝。
	3. 如果 `promise1` 被实现且 `onFulfilled` 不是一个函数，那么 `promise2` 必须以与 `promise1` 相同的值（value）被实现。
	4. 如果 `promise1` 被拒绝且 `onRejected` 不是一个函数，那么 `promise2` 必须以与 `promise1` 相同的原因（reason）被拒绝。


### 2\.3 Promise 解决过程


**Promise 解决过程** 是一种抽象操作，它接受一个 *promise* 和一个值作为输入，表示为 `[[Resolve]](promise, x)`。如果 `x` 是一个 *thenable*，则会尝试使 `promise` 采用 `x` 的状态，前提是 `x` 至少在某种程度上表现得像一个 *promise*。否则，它将用值 `x` 来实现 `promise`。


对 *thenable* 的这种处理方式使得不同的 *promise* 实现能够有效兼容，只要它们提供符合 Promises/A\+ 标准的 `then` 方法。但是也允许 Promises/A\+ 实现能够“接纳”那些具有合理 `then` 方法的非标准实现。


要实现 `[[Resolve]](promise, x)`，请按照以下步骤操作：


1. 如果 `promise` 和 `x` 指向同一个对象，则以 `TypeError` 拒绝 `promise` 作为原因。
2. 如果 `x` 是一个 *promise*，则采用其状态 \[[3\.4](https://github.com)]：
	1. 如果 `x` 是待定状态，`promise` 必须保持待定，直到 `x` 被实现或拒绝。
	2. 如果 `x` 被实现，使用相同的值实现 `promise`。
	3. 如果 `x` 被拒绝，使用相同的原因拒绝 `promise`。
3. 否则，如果 `x` 是一个对象或函数：
	1. 将 `then` 设置为 `x.then`。\[[3\.5](https://github.com):[veee加速器](https://youhaochi.com)]
	2. 如果检索属性 `x.then` 时抛出异常 `e`，则以 `e` 作为原因拒绝 `promise`。
	3. 如果 `then` 是一个函数，则以 `x` 作为 `this` 调用它，第一个参数为 `resolvePromise`，第二个参数为 `rejectPromise`，其中：
		1. 如果 `resolvePromise` 被调用并传入值 `y`，则运行 `[[Resolve]](promise, y)`。
		2. 如果 `rejectPromise` 被调用并传入原因 `r`，则以 `r` 拒绝 `promise`。
		3. 如果同时调用了 `resolvePromise` 和 `rejectPromise`，或对同一参数进行了多次调用，第一次调用优先，后续调用将被忽略。
		4. 如果调用 `then` 时抛出异常 `e`，
			1. 如果 `resolvePromise` 或 `rejectPromise` 已被调用，则忽略该异常。
			2. 否则，以 `e` 作为原因拒绝 `promise`。
	4. 如果 `then` 不是一个函数，则用 `x` 来实现 `promise`。
4. 如果 `x` 既不是对象也不是函数，则用 `x` 来实现 `promise`。


如果一个 *promise* 被一个循环的 *thenable* 链中的对象解决，而 `[[Resolve]](promise, thenable)` 的递归性质使得它被再次调用，按照上述算法将会导致无限递归。虽然算法并不强制要求检测这种递归，但我们鼓励实现者可以这样做。如果检测到存在循环，则应以一个语义清晰的 `TypeError` 拒绝 `promise`。\[[3\.6](https://github.com)]


## 3 引注


1. 这里的“平台代码”指的是引擎、环境和 promise 的实现代码。实际上，这一要求确保 `onFulfilled` 和 `onRejected` 在调用 `then` 的事件循环转之后异步执行，并且在一个新的调用栈中。这可以通过“宏任务”机制（如 [`setTimeout`](https://github.com) 或 [`setImmediate`](https://github.com)）或“微任务”机制（如 [`MutationObserver`](https://github.com) 或 [`process.nextTick`](https://github.com)）来实现。由于 promise 实现被视为平台代码，它本身可能包含一个任务调度队列或“跳板”，用于调用处理程序。
2. 也就是说，在严格模式下，`this` 的值将是 `undefined`；在非严格模式下，`this` 将指向全局对象。
3. 实现可以允许 `promise2 === promise1`，前提是该实现满足所有要求。每个实现应记录是否可以产生 `promise2 === promise1` 以及在什么条件下可以实现。
4. 通常，只有当 `x` 来自当前实现时，才能确定 `x` 是一个真正的 promise。这一条款允许使用特定于实现的方法来采用已知符合标准的 promises 的状态。
5. 这个过程首先存储对 `x.then` 的引用，然后测试该引用，最后调用该引用，避免了对 `x.then` 属性的多次访问。这种预防措施对于确保在访问器属性的情况下保持一致性非常重要，因为该属性的值可能在多次检索之间发生变化。
6. 实现不应对 thenable 链的深度设置任意限制，并假设超出该限制的递归将是无限的。只有真正的循环才应导致 `TypeError`；如果遇到无限的不同 thenable 链，则无限递归是正确的行为。


