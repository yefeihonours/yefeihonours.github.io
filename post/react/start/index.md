## 在线测试

在线测试地址:[https://jsfiddle.net/](https://jsfiddle.net/)

<!--more-->

## 组件们

每个组件都有一个功能叫 **render** ， 它可以高效的返回要映射到浏览器上 **HTML**。

## JSX

SX允许我们在Javascript中写HTML，而不是用HTML包含Javascript。

## 使用 JSX

如前面所说，JSX每一个组件都有一个 **render** 方法，这个方法产生一个”**ViewModel**(视图模型)” – 在返回 **HTML** 到组件之前，你可以将这个模型(**ViewModel**)的信息放入视图中，意味着你的 **HTML** 会根据这个模型动态改变(例如一个动态列表)。

一旦你完成了操作，就可以返回你想渲染(**render**)的东西，我们使用JSX来做是如此简单

## 将变量用在属性上

如果你想动态改变组件的样式类(或者任何其他的属性值),你可以使用变量. 不过你不能只是传进去一个变量名称，你还要将其用一对**大括弧**包起来, 这样JSX就能知道这是一个外部的变量了.


```
var ExampleComponent = React.createClass({
render: function () {
    var navigationClass = 'navigation';
    return (
        <div className={ navigationClass }>
            Hello World!
        </div>
        );
}
});

React.render(<ExampleContent />, document.body);
```

函数 **React.render()** 表示要渲染组件，**document.body** 表示在 body 上渲染组件

如果你希望的话，可以使用 render 函数许多次,

## 一个组件的基础

组件可以拥有其自己的**状态**。这使我们能够重复多次使用相同的组件但却让它们看起来完全不同，因为每个组件实例的状态是唯一的。

当你通过传递属性到一个组件时这些被称为属性。不要只局限于 **HTML** 属性，你可以传递任何你想传递的东西，并在组件内通过 **this.props** 访问它。这样使得我们能够重复使用相同的组件但传递一组不同的属性，比如组件的 **配置** 。

### 属性

在 HTML 节点上有 **className** 的属性。在组件内部，我们可以使用 **this.props.classname** 访问这个值


```
// 配置项
var navigationConfig = [];

var ExampleComponent = React.createClass({
	render: function () {
		return (
			<div className="navigation">
				Hello World!
			</div>
			);
	}
});
 
React.render(<ExampleComponent config={ navigationConfig } />, document.body);
```
如果现在能访问 this.props.config 的话，我们会收到一个空数组（**navigationConfig** 的值）。

### 状态

每一个组件都有其自己的**状态**。当要使用状态时，你要定义初始状态，让后才可以使用 **this.setState** 来更新状态。无论何时状态得到了更新，组件都会再一次调用 **render** 函数，拿新的值去替换或者改变之前渲染的值。

加一个 **getInitialState** 函数到类配置对象上，并返回带有我们想要的初始状态的对象。


```
...
var Navigation = React.createClass({
    getInitialState: function () {
        return {
            openDropdown: -1
        };
    },
    ...
    
});

React.render(<Navigation config={ navigationConfig } />, document.body);
```

你会发现我们将其设置成了 **-1** 。当我们准备去打开一个下拉菜单时，将使用状态里面的配置数组中的导航项的位置，而由于数组索引开始于**0**，我们就得使用 **-1** 来表示还没有指向一个导航项。

我们可以使用 **this.state** 来访问状态，因此如果我们去观察 **atthis.state.openDropdown**，应该会有 -1 被返回。

## 组件的生命周期

每个组件都有其 **生命周期** —这是一系列你可以在 **组件配置** 里定义的函数，它们将在部件生命周期内被调用。我们已经看过了 **getinitialstate** 一它只被调用一次，在组件被挂载时调用。

### componentWillMount

当组件要被挂载时这个函数被调用。这意味着我们可以在此运行组件功能必须的代码。为由于 **render** 在组件生命周期里被多次调用，我们一般会把只需要执行一次的代码放在这里，比如 XHR 请求。


```
var ExampleComponent = React.createClass({
    componentWillMount: function () {
        // xhr request here to get data
    },
    render: function () {
        // this gets called many times in a components life
        return (
            <div>
                Hello World!
            </div>
            );
    }
});
```

### componentDidMount

一旦你的组件已经运行了 **render** 函数，并实际将组件渲染到了 DOM 中，**componentDidMount** 就会被调用。我们可以在这儿做任何需要做的 DOM 操作，已经任何需要依赖于组件已经实际存在于 DOM 之后才能做的事情, 例如渲染一个图表。你可以通过调用 **this.getDOMNode** 在内部访问到 DOM 节点。


```
var ExampleComponent = React.createClass({
    componentDidMount: function () {
        var node = this.getDOMNode();
        // render a chart on the DOM node
    },
    render: function () {
        return (
            <div>
                Hello World!
            </div>
            );
    }
});
```
### componentWillUnmount

如果你准备吧组件从 DOM  移除时，这个函数将会被调用。这让我们可以在组件背后进行清理，比如移除任何我们已经绑定的事件监听器。

如果我们没有在自身背后做清理，而当其中一个事件被触发时，就会尝试去计算一个没有载入的组件，**React** 就会抛出一个错误。


```
var ExampleComponent = React.createClass({
    componentWillUnmount: function () {
        // unbind any event listeners specific to this component
    },
    render: function () {
        return (
            <div>
                Hello World!
            </div>
            );
    }
});
```

## 组件方法

React 也为组件提供了更方便我们工作的方法。它们会在组件的创建过程中被调用到。例如 **getInitialState**，能让我们定义一个默认的状态，这样我们不用担心在代码里面对状态项目是否存在做进一步的检查了。

### getDefaultProps

当我们创建组件时，可以按照我们的想法为组件的属性定义默认值。这意味着我们在调用组件时，如果给这些属性设置值，组件会有一个默认的"**配置**"，而我们就不用操心在下一行代码中检查属性这类事情了。

当你定义了组件时，这些默认的属性会被**缓存**起来，因而针对每个组件的实例它们都是一样的，并且不能被改变。对于导航组件，我们将配置指定为一个空的数组，因而就算没有传入配置，**render** 方法内也不会发生错误。


```
var Navigation = React.createClass({
    getInitialState: function () {
        ...
    },
    getDefaultProps: function () {
        return {
            config: []
        }
    },
    render: function () {
        ...
    }
});

React.render(...);
```

### propTypes

我们也可以随意为每一个**属性指定类型**。这对于我们检查和处理属性的意外赋值非常有用。如下面的**dropdown**，我们指定只有数组才能放入配置。


```
var Navigation = React.createClass({
    ...
    propTypes: {
        config: React.PropTypes.array
    },
    render: function () {
        ...
    }
});

React.render(...);
```

### mixins

这是一个独立于 React 的基本组件（只是一个对象类型的配置）。这意味着，如果我们有两个**功能类似**的组件，就可以共享一个配置了（**如果初始状态相同**）。我们可以抽象化地在 **mixin** 中建立一个方法，这样就不用把相同的代码写上两次了。

```
var ExampleMixin = {
    componentDidMount: function () {
        // bind some event listeners here
    },
    componentWillUnmount: function () {
        // unbind those events here!
    }
};

var ExampleComponent = React.createClass({
    mixins: [ExampleMixin],
    render: function () {
        ...
    }
});

var AnotherComponent = React.createClass({
    mixins: [ExampleMixin],
    render: function () {
        ...
    }
});

```

这样全部组件都有一样的 **componentDidMount** 和 **componentWillUnmount** 方法了，保存我们重写的代码。无论如何，你不能 **override**（覆盖）这些属性，如果这个属性是在mixin里设置的，它在这个组件中是不可覆盖的。

## 遍历循环

当我们有一个包含对象的数组，如何循环这个数组并渲染每一个对象到列表项中？JSX 允许你在任意 Javascript 文件中使用它，你可以映射这个数组并返回 JSX，然后使用 React 去渲染。


```
var navigationConfig = [
    {
        href: 'http://ryanclark.me',
        text: 'My Website'
    }
];
var Navigation = React.createClass({
    getInitialState: function () {
        return {
            openDropdown: -1
        };
    },
    getDefaultProps: function () {
        return {
            config: []
        }
    },
    propTypes: {
        config: React.PropTypes.array
    },
    render: function () {
        var config = this.props.config;
        var items = config.map(function (item) {
            return (
                <li className="navigation__item">
                    <a className="navigation__link" href={ item.href }>
                        { item.text }
                    </a>
                </li>
                );
        });
        return (
            <div className="navigation">
                { items }
            </div>
            );
    }
});
React.render(<Navigation config={ navigationConfig } />, document.body);
```
导航配置由数组和对象组成，包括一个指向超链接的 **href** 属性和一个用于显示的 **text** 属性。当我们映射的时候，它会一个个依次通过这个数组去取得对象。我们可以访问 **href** 和 **text**，并在 **HTML** 中使用。当返回列表时，这个数组里的列表项都将被替换，所以我们把它放入 **React** 中处理时它将知道怎么去渲染了！

## 混编

到目前位置，我们已经做到了所有下拉列表的展开。我们需要知道被下来的项目是哪个，我们将使用 **.children** 属性去遍历我们的 **navigationConfig** 数组。接下来，我们可以通过循环来操作下拉的子元素条目。


```
var navigationConfig = [
    {
        href: 'http://ryanclark.me',
        text: 'My Website',
        children: [
            {
                href: 'http://ryanclark.me/how-angularjs-implements-dirty-checking/',
                text: 'Angular Dirty Checking'
            },
            {
                href: 'http://ryanclark.me/getting-started-with-react/',
                text: 'React'
            }
        ]
    }
];
var Navigation = React.createClass({
    ...
    render: function () {
        var config = this.props.config;
        var items = config.map(function (item) {
            var children, dropdown;
            if (item.children) {
                children = item.children.map(function (child) {
                    return (
                        <li className="navigation__dropdown__item">
                            <a className="navigation__dropdown__link" 
                                href={ child.href }>
                                { child.text }
                            </a>
                        </li>
                    );
                });
                dropdown = (
                    <ul className="navigation__dropdown">
                        { children }
                    </ul>
                );
            }
            return (
                <li className="navigation__item">
                    <a className="navigation__link" href={ item.href }>
                        { item.text }
                    </a>
                    { dropdown }
                </li>
                );
        });
        return (
            <div className="navigation">
                { items }
            </div>
            );
    }
});
React.render(...);
```
但是我们还是能看见下来条目，尽管我们将 **openDropdown** 设置成为了 **-1** 。
我们可以通过在组件中访问 **this.state** ，来判断下拉是否被打开了，并且，我们可以为其添加一个新的 **css** 样式 **class** 来展现鼠标 **hover** 的效果。


```
var navigationConfig = [
    {
        href: 'http://ryanclark.me',
        text: 'My Website',
        children: [
            {
                href: 'http://ryanclark.me/how-angularjs-implements-dirty-checking/',
                text: 'Angular Dirty Checking'
            },
            {
                href: 'http://ryanclark.me/getting-started-with-react/',
                text: 'React'
            }
        ]
    }
];
var Navigation = React.createClass({
    getInitialState: function () {
        return {
            openDropdown: -1
        };
    },
    getDefaultProps: function () {
        return {
            config: []
        }
    },
    openDropdown: function (id) {
        this.setState({
            openDropdown: id
        });
    },
    closeDropdown: function () {
        this.setState({
            openDropdown: -1
        });
    },
    propTypes: {
        config: React.PropTypes.array
    },
    render: function () {
        var config = this.props.config;
        var items = config.map(function (item, index) {
            var children, dropdown;
            if (item.children) {
                children = item.children.map(function (child) {
                    return (
                        <li className="navigation__dropdown__item">
                            <a className="navigation__dropdown__link" 
                                href={ child.href }>
                                { child.text }
                            </a>
                        </li>
                    );
                });
                var dropdownClass = 'navigation__dropdown';
                if (this.state.openDropdown === index) {
                    dropdownClass += ' navigation__dropdown--open';
                }
                console.log(this.state.openDropdown, index);
                dropdown = (
                    <ul className={ dropdownClass }>
                        { children }
                    </ul>
                );
            }
            return (
                <li className="navigation__item" 
                onMouseOut={ this.closeDropdown } 
                onMouseOver={ this.openDropdown.bind(this, index) }>
                    <a className="navigation__link" href={ item.href }>
                        { item.text }
                    </a>
                    { dropdown }
                </li>
                );
        }, this);
        return (
            <div className="navigation">
                { items }
            </div>
            );
    }
});
React.render(...);
```
鼠标划过"**My Website**"，下拉即会展现。
在前面，我已经给每个列表项添加了鼠标事件。如你所见，我用的是 .bind (绑定) 调用，而非其它的方式调用。

这是因为，当用户的鼠标划出元素区域，我们并不用关注光标在哪里，所有我们需要知晓的是，将下拉关闭掉，所以我们可以将它的值设置成为 **-1** 。

但是，我们需要知晓的是当用户鼠标划入的时候哪个元素被下拉展开了，所以我们需要知道该参数(元素的索引)。

我们使用绑定的方式去调用而非简单的透过函数（**function**）去调用是因为我们需要通过 **React** 去调用。如果我们直接调用，那我们就需要一直调用，而不是在事件中调用他。

现在我们可以添加很多的条目到 **navigationConfig** 当中，而且我们也可以给他添加样式到下来功能当中。

