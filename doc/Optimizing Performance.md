# 性能优化

在更新UI时，React内部使用了多种技术最小化必要的DOM操作。对于大多数应用，使用React不需要额外的特定性能优化的情况下，就可以达到一个更快的用户交互。然而，下面有几种方式能够加快你的React应用。

## 使用生产环境

如果你在React应用中遇到性能的瓶颈，请确保你是在生产环境下测试。

默认地，React包含众多的帮助性的警告(warning)。这些警告在开发模式中非常有用。而然它们使得React体积庞大并性能下降，因此，你需要确保你是在生产模式下部署应用。

如果你不确定你部署的模式是否正确，你可以在Chrome中安装[React Developer Tools for Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)。如果你访问的网站是React生产模式，图标背景是深色:

![](https://facebook.github.io/react/img/docs/devtools-prod.png)

如果你访问的网站是React开发模式，图标的背景将会是红色:

![](https://facebook.github.io/react/img/docs/devtools-dev.png)

正常情况下，你会在开发过程中使用开发模式，当给用户部署应用使用生产模式。

### Create React App

如果你的工程使用[Create React App](https://github.com/facebookincubator/create-react-app)，运行:

```
npm run build
```


这将会在你的工程中`build/`目录下创建生产模式的应用。

记住，这是指针对于部署产品。对于普通的开发者，使用 `npm start`。

### Single-File Builds

我们提供了生产模式的React和React DOM的文件:

```html
<script src="https://unpkg.com/react@15/dist/react.min.js"></script>
<script src="https://unpkg.com/react-dom@15/dist/react-dom.min.js"></script>
```

需要记住的是以`.min.js`结尾的React文件仅适用于生产模式。

### Brunch

如果使用的高效地Brunch构建，安装[uglify-js-brunch](https://github.com/brunch/uglify-js-brunch)插件:

```
# 使用npm
npm install --save-dev uglify-js-brunch

# 使用yarn
yarn add --dev uglify-js-brunch
```

然后，通过给`build`命令添加`-p`来创建生产模式应用。在开发模式下不要传递`-p`标志或者使用上述插件，因为其隐藏了有用的React警告(warning)并是构建速度降低。

### Browserify

对于使用的高效地Browserify构建，安装下列插件:

```
# 使用npm
npm install --save-dev bundle-collapser envify uglify-js uglifyify

# 使用Yarn
yarn add --dev bundle-collapser envify uglify-js uglifyify
```

为了创建生产模式的应用，确实你添加了下列的转化规则(顺序很重要):

* [`envify`](https://github.com/hughsk/envify)确保能设置正确的构建环境。全局安装(`-g`)。
* [`uglifyify`](https://github.com/hughsk/uglifyify)能移除开发环境引入的文件。全局安装(`-g`)。
* [`bundle-collapser`](https://github.com/substack/bundle-collapser)插件可以用数字替换模块ID。
* 最后，使用[`uglify-js`](https://github.com/mishoo/UglifyJS2)压缩打包结果([查看为什么](https://github.com/hughsk/uglifyify#motivationusage))

例如:

```
browserify ./index.js \
  -g [ envify --NODE_ENV production ] \
  -g uglifyify \
  -p bundle-collapser/plugin \
  | uglifyjs --compress --mangle > ./bundle.js
```

>**注意:**
>包名为`uglify-js`, 但提供名为`uglifyjs`。<br>
>并不是排版错误

### Rollup

对于使用的高效地Rollupy构建，安装下列插件:

```
# 如果使用的是npm
npm install --save-dev rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-uglify

# 如果使用的yarn
yarn add --dev rollup-plugin-commonjs rollup-plugin-replace rollup-plugin-uglify
```

为了创建生产模式的应用，确实你添加了下列插件(顺序很重要):

* [`replace`](https://github.com/rollup/rollup-plugin-replace) 插件确保构建正确的构建环境。
* [`commonjs`](https://github.com/rollup/rollup-plugin-commonjs) 插件使得Rollup支持CommonJS。
* [`uglify`](https://github.com/TrySound/rollup-plugin-uglify) 插件压缩最终打包的文件。

```js
plugins: [
  // ...
  require('rollup-plugin-replace')({
    'process.env.NODE_ENV': JSON.stringify('production')
  }),
  require('rollup-plugin-commonjs')(),
  require('rollup-plugin-uglify')(),
  // ...
]
```

完整的例子查看[gist](https://gist.github.com/Rich-Harris/cb14f4bc0670c47d00d191565be36bf0)

记住你仅需要在生产模式下使用，你不应该在开发模式下使用`uglify`或`replace`插件，因为它隐藏了有用的React警告并使得构建速度变慢。

### Webpack

>**注意:**
>
>如果你使用的是Create React App, 查看[这个例子](#create-react-app).<br>
>这个小节针对于你直接配置Webpack

对于使用最高效地Rollupy构建，确保在生产配置下添加下面插件:

```js
new webpack.DefinePlugin({
  'process.env': {
    NODE_ENV: JSON.stringify('production')
  }
}),
new webpack.optimize.UglifyJsPlugin()
```

更多的信息可以了解[Webpack文档](https://webpack.js.org/guides/production-build/)

记住你仅需要在生产模式下使用。你不应该在开发模式下使用`UglifyJsPlugin`或`DefinePlugin`插件，因为它隐藏了有用的React警告并使得构建速度变慢。

## 使用Chrome Timeline分析组件性能

在开发模式中，你可以在支持相关功能的浏览器中使用性能工具来可视化组件安装(mount)，更新(update)和卸载(unmount)的各个过程。例如:

<center><img src="https://facebook.github.io/react/img/blog/react-perf-chrome-timeline.png" width="651" height="228" alt="React components in Chrome timeline" /></center>

在Chrome操作如下:

1. 通过添加`?react_perf`查询字段加载你的应用(例如：`http://localhost:3000/?react_perf`)

2. 打开Chrome DevTools **[Timeline](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/timeline-tool)** 并点击**Record**。

3. 执行你想要分析的操作，不要超过20秒，否则Chrome可能会挂起。

4. 停止记录。

5. 在**User Timing**下，React事件将会分组列出。

注意，上述数据是相对的，组件会在生产环境中有更好的性能。然而，这对你分析由于错误导致不相关的组件的更新、分析组件更新的深度和频率很有帮助。

目前Chrome，Edge和IE支持该特性，但是我们使用了标准的 [User Timing API](https://developer.mozilla.org/en-US/docs/Web/API/User_Timing_API),因此我们期待将来会有更多的浏览器支持。

## 避免Reconciliation

React创建和维护了渲染UI的内部状态。其包括了组件返回的React元素。这些内部状态使得React只有在必要的情况下才会创建DOM节点和访问存在DOM节点，因为对JavaScript对象的操作是比DOM操作更快。这被称为"虚拟DOM"，React Native也基于上述原理。

当组件的`props`和`state`更新时,React通过比较新返回的元素和之前渲染的元素来决定是否有必要更新DOM元素。如果二者不相等，则更新DOM元素。

在部分场景下，组件可以通过重写生命周期函数`shouldComponentUpdate`来优化性能。`shouldComponentUpdate`函数会在重新渲染流程前触发。`shouldComponentUpdate`的默认实现中返回的是`true`，使得React执行更新操作。

```javascript
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

如果你的组件在部分场景下不需要更行，你可以在`shouldComponentUpdate`返回`false`来跳过整个渲染流程(包括调用`render`和之后流程)。

## shouldComponentUpdate

下面有一个组件子树，其中`SCU`代表`shouldComponentUpdate`函数返回结果。`vDOMEq`代表渲染的React元素是否相等。最后，圆圈内的颜色代表组件是否需要reconcile(译者注:reconcile代表React在每次需要渲染时，会先比较当前DOM内容和待渲染内容的差异， 然后再决定如何最优地更新DOM)

<figure><img src="https://facebook.github.io/react/img/docs/should-component-update.png" /></figure>

因为以C2为根节点的子树`shouldComponentUpdate`返回的是`false`,React不会尝试重新渲染C2,并且也不会尝试调用C4和C5的`shouldComponentUpdate`。

对于C1和C3,`shouldComponentUpdate`返回`false`,所以React需要向下遍历，对于C6,`shouldComponentUpdate`返回`false`,并且需要渲染的元素不相同，因此React需要更新DOM节点。

最后一个值得注意的例子是C8.React必须渲染这个组件，但是由于返回的React元素与之前渲染的元素相比是相同的，因此不需要更新DOM节点。

注意，React仅仅需要修改C6的DOM，这是必须的。对于C8来讲，通过比较渲染元素被剔除，对于C2子树和C7,因为`shouldComponentUpdate`被剔除，甚至都不需要比较React元素，也不会调用`render`方法。

## 例子

仅当`props.color`和`state.count`发生改变时，组件需要更新，你可以通过`shouldComponentUpdate`函数设置：

```javascript
class CounterButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

在上面的代码中，`shouldComponentUpdate`函数仅仅检查`props.color` 或者 `state.count`是否发生改变。如果这些值没有发生变化，则组件不会进行更新。如果你的组件更复杂，你可以使用类似于对`props`和`state`的所有属性进行"浅比较"这种模式来决定组件是否需要更新。这种模式非常普遍，因此React提供了一个helper实现上面的逻辑：继承`React.PureComponent`。因此，下面的代码是一种更简单的方式实现了相同的功能：

```js
class CounterButton extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

大多数情况下，你可以使用`React.PureComponent`而不是自己编写`shouldComponentUpdate`。但`React.PureComponent`仅会进项浅比较，因此如果在props和state会突变(译者注：就是引用不发生变化，但指向的内容发生变化)导致浅比较失败的情况下就不能使用`React.PureComponent`。

如果props和state属性存在更复杂的数据结构，这可能是一个问题。例如，我们编写一个`ListOfWords`组件展现一个以逗号分隔的单词列表，在父组件`WordAdder`，当你点击一个按钮时会给列表添加一个单词。下面的代码是**不正确**的：

```javascript
class ListOfWords extends React.PureComponent {
  render() {
    return <div>{this.props.words.join(',')}</div>;
  }
}

class WordAdder extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      words: ['marklar']
    };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // This section is bad style and causes a bug
    const words = this.state.words;
    words.push('marklar');
    this.setState({words: words});
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick} />
        <ListOfWords words={this.state.words} />
      </div>
    );
  }
}
```

问题是`PureComponent`只进行在旧的`this.props.words`与新的`this.props.words`之间进行前比较。因此在`WordAdder`组件中`handleClick`的代码会突变`words`数组。虽然数组中实际的值发生了变化，但旧的`this.props.words`和新的`this.props.words`值是相同的，即使`ListOfWords`需要渲染新的值，但是还是不会进行更新。

## 不可变数据的力量

避免这类问题最简单的方法是不要突变(mutate)props和state的值。例如，上述`handleClick`方法可以通过使用`concat`重写:

```javascript
handleClick() {
  this.setState(prevState => ({
    words: prevState.words.concat(['marklar'])
  }));
}
```

ES6对于数组支持[展开语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator) ，使得解决上述问题更加简单。如果你使用的是Create React App，默认支持该语法。

```js
handleClick() {
  this.setState(prevState => ({
    words: [...prevState.words, 'marklar'],
  }));
};
```

你可以以一种简单的方式重写上述代码，使得改变对象的同时不会突变对象，例如，如果有一个`colormap`的对象并且编写一个函数将`colormap.right`的值改为`blue`：

```js
function updateColorMap(colormap) {
  colormap.right = 'blue';
}
```

在不突变原来的对象的条件下实现上面的要求，我们可以使用[Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)方法：

```js
function updateColorMap(colormap) {
  return Object.assign({}, colormap, {right: 'blue'});
}
```

`updateColorMap`方法返回一个新的对象，而不是修改原来的对象。`Object.assign`属于ES6语法，需要polyfill。

JavaScript提案添加了[对象展开符](https://github.com/sebmarkbage/ecmascript-rest-spread)，能够更简单地更新对象而不突变对象。

```js
function updateColorMap(colormap) {
  return {...colormap, right: 'blue'};
}
```
如果你使用的是Create React App，`Object.assign`和对象展开符默认都是可用的。

## 使用Immutable 数据结构

[Immutable.js](https://github.com/facebook/immutable-js) 是解决上述问题的另外一个方法，其提供了通过结构共享实现(Structural Sharing)地不可变的(Immutable)、持久的(Persistent)集合:

* *不可变(Immutable)*: 一个集合一旦创建，在其他时间是不可更改的。
* *持久的(Persistent)*: 新的集合可以基于之前的结合创建并产生突变，例如：set。原来的集合在新集合创建之后仍然是可用的。
* *结构共享(Structural Sharing)*: 新的集合尽可能通过之前集合相同的结构创建，最小程度地减少复制操作来提高性能。

不可变性使得追踪改变非常容易。改变会产生新的对象，因此我们仅需要检查对象的引用是否改变。例如，下面是普通的JavaScript代码：

```javascript
const x = { foo: 'bar' };
const y = x;
y.foo = 'baz';
x === y; // true
```
虽然`y`被编辑了，但是因为引用的是相同的对象`x`,所以比较返回`true`。

```javascript
const SomeRecord = Immutable.Record({ foo: null });
const x = new SomeRecord({ foo: 'bar' });
const y = x.set('foo', 'baz');
x === y; // false
```

在这个例子中，因为当改变x时返回新的引用，我们可以确信地判定`x`被改变了。

其他两个可以帮助我们使用不可变数据的库分别是:[seamless-immutable](https://github.com/rtfeldman/seamless-immutable)和[immutability-helper](https://github.com/kolodny/immutability-helper).

不可变数据提供了一种更简单的方式来追踪对象的改变，这正是我们实现`shouldComponentUpdate`所需要的。这将会提供可观的性能提升。
