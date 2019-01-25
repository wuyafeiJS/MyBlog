---
title: 为何要写super(props)?
data: '2018-11-30'
spoiler: 译自 Dan Abramov 的博客
---

**其实这个疑问对于你正常的使用 _React_ 来说并不重要，但如果你想深挖 _React_ 的工作原理，那么你会发现它的有趣之处**

第一点

---

我写`super(props)`的次数比我想到的还多。

```jsx{3}
class Checkbox extends React.Component {
  constructor(props) {
    super(props)
    this.state = { isOn: true }
  }
}
// ...
```

当然，新的[class fields 提案](https://github.com/tc39/proposal-class-fields)让我们避免了这段代码：

```jsx
class Checkbox extends React.Component {
  state = {
    isOn: true,
  }
}
```

这种语法在 React 0.13 版本得到支持。而且这种写法也更加符合人体工学。

让我们继续回到这个例子，它用到了 es6 的特性：

```jsx{3}
class Checkbox extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      isOn: true,
    }
  }
}
```

那么问题来了，**我们为啥要调用 super？ 我们可以不用它吗。如果我们调用它，但是不加 props 会发生什么？它还有其他的参数吗？** 让我们来探索一下。

---

在 js 中，`super` 关联到父级 class 的 constructor。（在我们上面的例子中，它指向`React.Component`）

重点是，我们可以在调用父级`class constructor`后使用 `this`。也就是在`super()`之前是不能用`this`的：

```jsx
class Checkbox extends React.Components {
  constructor(props) {
    //🔴 这里还不能使用`this`
    super(props)
    // 这里就可以使用了
    this.state = { isOn: true }
  }
}
```

我们有个很好的理由去解释为啥 Javascript 强制 调用父级 constructor 要在使用 this 前执行。我们想想 class 的层级：

```jsx
class Person {
  constructor(name) {
    this.name = name
  }
}

class PolitePerson extends Person {
  constructor(name) {
    this.greetColleagues() // 🔴 这是不被允许的
    super(name)
  }
  greetColleagues() {
    alert('Good morning folks')
  }
}
```

想象一下，如果我们可以在`super`之前调用 this。一段时间后，我们可能需要修改下`greetColleagues`，如在这个函数里面调用`this.name`:

```jsx
  greetColleagues() {
    alert('Good morning folks!');
    alert('My name is ' + this.name + ', nice to meet you!');
  }
```

但是我们忘了 `this.greetColleagues()` 是在 super 之前调用了，这时候`this.name`压根不存在。如你所见，这种代码是很蠢的。

为了避免这个陷阱， **JS 强制在调用`this`前先得调用`super`.** 这个限制同样应用到了 React Components:

```jsx
  constructor(props) {
    super(props);
    // ✅ Okay to use `this` now
    this.state = { isOn: true };
  }
```

这段代码给我们留下了另一个疑问： 为什么让 props 作为 super 的参数？

---

你可能会想，让`props`作为`super`的参数之所以需要，是因为这样 基础的`React.Component` 的 `constructor` 能够初始化 `this.props`，像这样：

```jsx
// React的内部操作， React.Component对象
class Component {
  constructor(props) {
    this.props = props
    // ...
  }
}
```

你如果这样想，那么你离真相不远了，这就是[React 源码的操作](https://github.com/facebook/react/blob/1d25aa5787d4e19704c049c3cfa985d3b5190e0d/packages/react/src/ReactBaseClasses.js#L22).

但是有些时候，你调用`super()`但是不传入`props`，你依旧可以在`render`或者其他方法里面使用`this.props`。

那么它是如何工作的呢？ 这就引出了 **当调用`constructor`之后，React 也会在实例上立即分配`props`**

```jsx
// React 内部
const instance = new YourComponent(props)
instance.props = props
```

所以，就算你在写`super()`的时候忘了传入 props，React 依旧会在之后设置它们。下面说下这么做的原因。

React 新增了对 classes 的支持，它不仅仅只是为了支持 es6 的 classes 。我们的目标是支持尽可能多的 class 抽象。我们[并不清楚](https://reactjs.org/blog/2015/01/27/react-v0.13.0-beta-1.html#other-languages) ClojureScript, CoffeeScript, ES6, Fable, Scala.js, TypeScript, 和一些其他的语言 它们对组件的定义，哪一种相对来说会比较成功。所以 React 故意忽略了是否必须调用 super()这一问题 - 尽管 es6 是必须调用的。

那么，这是否意味着你只需写`super()`来代替`super(props)`呢？

**答案可能是否定的，因为这里仍有疑问** 当然，React 会在执行 constructor 之后立刻分发 `this.props`。 但是`this.props`在调用`super` 和 执行完 constructor 之间 仍然是 undefined：

```jsx
// React 内部
class Component {
  constructor(props) {
    this.props = props
    // ...
  }
}

// 你的代码
class Button extends React.Component {
  constructor(props) {
    super() // 如果这里忘了加props
    console.log(props) // {}
    console.log(this.props) // undefined
  }
}
```

如果在 constructor 中某个方法调用了`this.props`，上面的写法就会给 debug 带来很大的挑战。
**这就是我为啥建议每次都得写`super(props)`的原因，虽然它不是必须的：**

```jsx
class Button extends React.Component {
  constructor(props) {
    super(props) // ✅ We passed props
    console.log(props) // ✅ {}
    console.log(this.props) // ✅ {}
  }
  // ...
}
```

这样就能保证在 constructor 诞生之前就设置了`this.props`。

---

长期以来，React 使用者可能会有一点疑惑。

大家可能注意到，在 classes 中使用 `context api` (不管是旧的 `contextTypes`还是在 React16.6 中新增的新的`contextType` API), `context` 都是作为 constructor 的第二个参数。

所以为啥我们不提倡像 `super(props, context)` 这种写法呢？ 其实这种写法也没问题，但是`context`使用量较少所以这样的陷阱遇到的不多。

**随着 class fields 提案的诞生，这个陷进已经几乎消失。** 现在我们压根不需要写 constructor, 所有的参数都会被自动添加，所以现在我们直接这样写： `state={}`，然后可以在里面直接调用`this.props` 或 `this.context`。

最后，随着 Hooks 的诞生， 我们现在甚至都不要`super` 和 `this`。 不过这就是下个话题了。
