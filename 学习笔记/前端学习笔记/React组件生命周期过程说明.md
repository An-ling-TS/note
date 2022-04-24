# React组件生命周期过程说明

[toc]

# 实例化

首次实例化

- getDefaultProps
- getInitialState
- componentWillMount
- render
- componentDidMount

实例化完成后的更新

- getInitialState
- componentWillMount
- render
- componentDidMount

# 存在期

组件已存在时的状态改变

- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate
- render
- componentDidUpdate

# 销毁&清理期

- componentWillUnmount

# 说明

生命周期共提供了10个不同的API。

## 1.getDefaultProps

作用于组件类，只调用一次，返回对象用于设置默认的`props`，对于引用值，会在实例中共享。

## 2.getInitialState

作用于组件的实例，在实例创建时调用一次，用于初始化每个实例的`state`，此时可以访问`this.props`。

## 3.componentWillMount

在完成首次渲染之前调用，此时仍可以修改组件的state。

## 4.render

必选的方法，创建虚拟DOM，该方法具有特殊的规则：

- 只能通过`this.props`和`this.state`访问数据
- 可以返回`null`、`false`或任何React组件
- 只能出现一个顶级组件（不能返回数组）
- 不能改变组件的状态
- 不能修改DOM的输出

## 5.componentDidMount

真实的DOM被渲染出来后调用，在该方法中可通过`this.getDOMNode()`访问到真实的DOM元素。此时已可以使用其他类库来操作这个DOM。

在服务端中，该方法不会被调用。

## 6.componentWillReceiveProps

组件接收到新的`props`时调用，并将其作为参数`nextProps`使用，此时可以更改组件`props`及`state`。

```
    componentWillReceiveProps: function(nextProps) {
        if (nextProps.bool) {
            this.setState({
                bool: true
            });
        }
    }
```

## 7.shouldComponentUpdate

组件是否应当渲染新的`props`或`state`，返回`false`表示跳过后续的生命周期方法，通常不需要使用以避免出现bug。在出现应用的瓶颈时，可通过该方法进行适当的优化。

在首次渲染期间或者调用了`forceUpdate`方法后，该方法不会被调用

## 8.componentWillUpdate

接收到新的`props`或者`state`后，进行渲染之前调用，此时不允许更新`props`或`state`。

## 9.componentDidUpdate

完成渲染新的`props`或者`state`后调用，此时可以访问到新的DOM元素。

## 10.componentWillUnmount

组件被移除之前被调用，可以用于做一些清理工作，在`componentDidMount`方法中添加的所有任务都需要在该方法中撤销，比如创建的定时器或添加的事件监听器。



# 组件 API

## ReactComponent

React 组件实例在渲染的时候创建。这些实例在接下来的渲染中被重复使用，可以在组件方法中通过 `this` 访问。唯一一种在 React 之外获取 React 组件实例句柄的方式就是保存 `React.render` 的返回值。在其它组件内，可以使用 refs 得到相同的结果。

### setState

```
setState(object nextState[, function callback])
```

合并 nextState 和当前 state。这是在事件处理函数中和请求回调函数中触发 UI 更新的主要方法。另外，也支持可选的回调函数，该函数在 `setState` 执行完毕并且组件重新渲染完成之后调用。

> 注意：
>
> 绝对不要直接改变 `this.state`，因为在之后调用 `setState()` 可能会替换掉你做的更改。把 `this.state` 当做不可变的。
>
> `setState()` 不会立刻改变 `this.state`，而是创建一个即将处理的 state 转变。在调用该方法之后获取 `this.state` 的值可能会得到现有的值，而不是最新设置的值。
>
> 不保证 `setState()` 调用的同步性，为了提升性能，可能会批量执行 state 转变和 DOM 渲染。
>
> `setState()` 将总是触发一次重绘，除非在 `shouldComponentUpdate()` 中实现了条件渲染逻辑。如果使用可变的对象，但是又不能在 `shouldComponentUpdate()` 中实现这种逻辑，仅在新 state 和之前的 state 存在差异的时候调用 `setState()` 可以避免不必要的重新渲染。

### replaceState

```
replaceState(object nextState[, function callback])
```

类似于 `setState()`，但是删除之前所有已存在的 state 键，这些键都不在 nextState 中。

### forceUpdate()

```
forceUpdate([function callback])
```

如果 `render()` 方法从 `this.props` 或者 `this.state` 之外的地方读取数据，你需要通过调用 `forceUpdate()` 告诉 React 什么时候需要再次运行 `render()`。如果直接改变了 `this.state`，也需要调用 `forceUpdate()`。

调用 `forceUpdate()` 将会导致 `render()` 方法在相应的组件上被调用，并且子级组件也会调用自己的 `render()`，但是如果标记改变了，那么 React 仅会更新 DOM。

通常情况下，应该尽量避免所有使用 `forceUpdate()` 的情况，在 `render()` 中仅从 `this.props` 和 `this.state` 中读取数据。这会使应用大大简化，并且更加高效。

```ts
class App extends React.Component {
	constructor(props) {
		super(props);
		console.log("constructor")

		this.onClickHandler = this.onClickHandler.bind(this);
	}
	componentWillMount() {
		console.log("componentWillMount")
	}
	componentDidMount() {
		console.log("componentDidMount")
	}
	componentWillUnmount() {
		console.log("componentWillUnmount")
	}
	componentWillReceiveProps() {
		console.log("componentWillReceiveProps")
	}
	shouldComponentUpdate() {
		console.log("shouldComponentUpdate")
		return true
	}
	componentWillUpdate() {
		console.log("componentWillUpdate")
	}
	componentDidUpdate() {
		console.log("componentDidUpdate")
	}

	onClickHandler() {
		console.log("onClickHandler")
		this.forceUpdate();
	}

	render() {
		console.log("render")
		return (
			<button onClick={this.onClickHandler}> click here </button>
		);
	}
}

ReactDOM.render(<App />,
	document.getElementById("react-container")
);

```



### getDOMNode

```
DOMElement getDOMNode()
```

如果组件已经挂载到了 DOM 上，该方法返回相应的本地浏览器 DOM 元素。从 DOM 中读取值的时候，该方法很有用，比如获取表单字段的值和做一些 DOM 操作。当 `render` 返回 `null` 或者 `false` 的时候，`this.getDOMNode()` 返回 `null`。

### isMounted()

```
bool isMounted()
```

如果组件渲染到了 DOM 中，`isMounted()` 返回 true。可以使用该方法保证 `setState()` 和 `forceUpdate()` 在异步场景下的调用不会出错。

### setProps

```
setProps(object nextProps[, function callback])
```

当和一个外部的 JavaScript 应用集成的时候，你可能想给一个用 `React.render()` 渲染的组件打上改变的标记。

尽管在同一个节点上再次调用 `React.render()` 来更新根组件是首选的方式，也可以调用 `setProps()` 来改变组件的属性，触发一次重新渲染。另外，可以传递一个可选的回调函数，该函数将会在 `setProps` 完成并且组件重新渲染完成之后调用。

> 注意：
>
> When possible, the declarative approach of calling `React.render()` again is preferred; it tends to make updates easier to reason about.  (There's no significant performance difference between the two  approaches.)
>
> 刚方法仅在根组件上面调用。也就是说，仅在直接传给 `React.render()` 的组件上可用，在它的子级组件上不可用。如果你倾向于在子组件上使用 `setProps()`，不要利用响应式更新，而是当子组件在 `render()` 中创建的时候传入新的 prop 到子组件中。

### replaceProps

```
replaceProps(object nextProps[, function callback])
```

类似于 `setProps()`，但是删除所有已存在的 props，而不是合并新旧两个 props 对象。



 React对底层的代码作了封装，在大多数情况下，我们不需要直接去操作DOM。但是有时候我们还是需要使用到底层的代码的，比如输入框获取焦点，这个时候可以通过第三方的类库或者React提供的API实现。

# 虚拟DOM

React之所以快，是因为它不直接操作DOM。React将DOM结构存储在内存中，然后同render()的返回内容进行比较，计算出需要改动的地方，最后才反映到DOM中。
此外，React实现了一套完整的事件合成机制，能够保持事件冒泡的一致性，跨浏览器执行。甚至可以在IE8中使用HTML5的事件。
大部分情况下，我们都是在构建React的组件，也就是操作虚拟DOM。但是有时候我们需要访问底层的API，可能或通过使用第三方的插件来实现我们的功能，如jQuery。React也提供了接口让我们操作底层API。

# Refs和findDOMNode()

为了同浏览器交互，我们有时候需要获取到真实的DOM节点。我们可以通过调用React的React.findDOMNode(component)获取到组件中真实的DOM。

> React.findDOMNode()只在mounted组件中调用，mounted组件就是已经渲染在浏览器DOM结构中的组件。如果你在组件的render()方法中调用React.findDOMNode()就会抛出异常。

看官方的示例：

```
var MyComponent = React.createClass({
  handleClick: function() {
    // Explicitly focus the text input using the raw DOM API.
    React.findDOMNode(this.refs.myTextInput).focus();
  },
  render: function() {
    // The ref attribute adds a reference to the component to
    // this.refs when the component is mounted.
    return (
      <div>
        <input type="text" ref="myTextInput" />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.handleClick}
        />
      </div>
    );
  }
});
React.render(
  <MyComponent />,
  document.getElementById('example')
);
```

# 组件的生命周期

组件的生命周期主要由三个部分组成：

- Mounting：组件正在被插入DOM中
- Updating：如果DOM需要更新，组件正在被重新渲染
- Unmounting：组件从DOM中移除

React提供了方法，让我们在组件状态更新的时候调用，will标识状态开始之前，did表示状态完成后。例如componentWillMount就表示组件被插入DOM之前。

## Mounting

- getInitialState()：初始化state
- componentWillMount()：组件被插入DOM前执行
- componentDidMount()：组件被插入DOM后执行

## Updating

- componentWillReceiveProps(object nextProps):组件获取到新的属性时执行，这个方法应该将this.props同nextProps进行比较，然后通过this.setState()切换状态
- shouldComponentUpdate(object nextProps, object  nextState):组件发生改变时执行，应该将this.props和nextProps、this.stats和nextState进行比较，返回true或false决定组件是否更新
- componentWillUpdate(object nextProps, object nextState)：组件更新前执行，不能在此处调用this.setState()。
- componentDidUpdate(object prevProps, object prevState)：组件更新后执行

## Unmounting

- componentWillUnmount()：组件被移除前执行

## Mounted Methods

- findDOMNode()：获取真实的DOM
- forceUpdate()：强制更新