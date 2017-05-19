# 二.这一切的开始 React.js

> path: src/isomorphic/

当找到了新世界的大门后，我们当然要看一下这里面包含了一些什么

```js
var React = {
  // Modern

  Children: {
    map: ReactChildren.map,
    forEach: ReactChildren.forEach,
    count: ReactChildren.count,
    toArray: ReactChildren.toArray,
    only: onlyChild,
  },

  Component: ReactBaseClasses.Component,
  PureComponent: ReactBaseClasses.PureComponent,

  createElement: createElement,
  cloneElement: cloneElement,
  isValidElement: ReactElement.isValidElement,

  // TODO (bvaughn) Remove these getters before 16.0.0
  PropTypes: ReactPropTypes,
  checkPropTypes: checkPropTypes,
  createClass: createReactClass,

  // Classic

  createFactory: createFactory,
  createMixin: createMixin,

  // This looks DOM specific but these are actually isomorphic helpers
  // since they are just generating DOM strings.
  DOM: ReactDOMFactories,

  version: ReactVersion,

  __SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED: {
    ReactCurrentOwner: require('ReactCurrentOwner'),
  },
};
```

我们可以看到在这里，将React中的所有API都暴露了出来，其中当然包括了我们很熟悉的createElement、cloneElement、Component等API。那我们当然先看看Component，我们都已经写了很多很多的React组件了，在每次写组件的时候，我们都会使用

```
class MyComponent extends React.Component
```

那到底我们的组件从React.Component中继承了什么呢，我们从require的ReactBaseClasses模块中找到Component。

在这里稍微简单的说一下react的模块化，在我们见到代码中

```
var ReactBaseClasses = require('ReactBaseClasses');
var ReactChildren = require('ReactChildren');
var ReactDOMFactories = require('ReactDOMFactories');
var ReactElement = require('ReactElement');
var ReactPropTypes = require('ReactPropTypes');
var ReactVersion = require('ReactVersion');
```

我们看到这样的require，我们很下意识的想到这模块当然在node\_modules中，但是经过一番的查找，才发现在这里react用了一个特殊的模块化方法，在react的文件编译后，会变成一个扁平化的结构，而在这里的`require('ReactBaseClasses')`则会被编译成`require('./ReactBaseClasses')`，所以我们很快就能发现，其实每一个模块都是一个文件，并且文件名和模块名是一致的。你要找的模块其实就是找同名的文件即可。

当我们知道了这样的模块化规则之后，也很快找到了ReactBaseClasses.js文件

> path: src/isomorphic/modern/class/ReactBaseClasses.js

```
function ReactComponent(props, context, updater) {
  this.props = props;
  this.context = context;
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}

ReactComponent.prototype.setState = function(partialState, callback) {
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};

ReactComponent.prototype.forceUpdate = function(callback) {
  this.updater.enqueueForceUpdate(this, callback, 'forceUpdate');
};

```

通过上面的代码，我们可以看到ReactComponent是一个构造函数，并且里面有setState和forceUpdate这两个方法。

