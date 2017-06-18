# 虚拟DOM\(Virtual DOM\)

我们来看看JSX这个语法到底会被编译成什么

```
<div className="div">
    <MyComponent className="haha" style={{backgroundColor:"red"}} >Hello MyComponent</MyComponent>
</div>
```

通过编译，以上代码会变成

```js
React.createElement('div',{
    className:"div"
},React.createElement(MyComponent,{
    className:'haha',
    style:{
        backgroundColor:'red'
    }
},"Hello MyComponent"))
```

看得出来，在我们平时写着类似HTML的结构的JSX语法被转换成React.createElement的方法，而通过React.createElement方法执行后返回的则是一个ReactElement。那让我们来看一下React.createElemen这个方法到底做了什么

#### CreateElement

> path:src\isomorphic\classic\element\ReactElement.js

首先来看一下，createElement的方法，最后返回的是一个ReactElement的方法

```js
ReactElement.createElement = function(type, config, children) {
  var propName;

  // Reserved names are extracted
  var props = {};

  var key = null;
  var ref = null;
  var self = null;
  var source = null;

  if (config != null) {
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    if (hasValidKey(config)) {
      key = '' + config.key;
    }

    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // Remaining properties are added to a new props object
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }

  // Children can be more than one argument, and those are transferred onto
  // the newly allocated props object.
  var childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    var childArray = Array(childrenLength);
    for (var i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    props.children = childArray;
  }

  // Resolve default props
  if (type && type.defaultProps) {
    var defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }

  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
};
```

我们来一步一步来看这个方法，这个方法接受了三个type，config， children。在一开始第一个判断当`config !== null` 的时候处理了一下config中的ref、key、props。在第二个判断则是获取第三个参数开始的children，在这里可以发现children可以为一个，同时也可以是多个，当多个的时候，就会用一个childrenArray将多个children放进去。最后的判断处理defaultProps。在这个方法最后返回了一个ReactElement的方法。

#### ReactElement

```js
var ReactElement = function(type, key, ref, self, source, owner, props) {
  var element = {
    // This tag allow us to uniquely identify this as a React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner,
  };
  return element;
};
```

在这里可以看出来ReactElement其实就是一个对象，而这个就是react中的虚拟DOM，在其中里面包括了`type,$$typeof,key,ref,__owner`的属性。

首先来看看`$$typeof`

```js
var REACT_ELEMENT_TYPE =
  (typeof Symbol === 'function' && Symbol.for && Symbol.for('react.element')) ||
  0xeac7;
```

这其实是一个ReactElement的标识，来判断这个对象是否是一个ReactElement的，然后\_owner则是描述当前节点。

现在有了ReactElement了，那我们需要将这一个ReactElement渲染到浏览器上，自然需要一个render方法。

#### render

在我们使用react的时候，会发现我们总共会写两种render，一种是在组件中写的render方法，第二个则是在ReactDom.render方法，首先我们来看一下第一种，第一种的本质其实是返回了我们React.createElement得到的ReactElement。而第二种则是将这一个ReactElement渲染到浏览器的方法。因此我们先找到ReactDom

> path:src\renderers\dom\ReactDOM.js

在这文件中，我们看到了一个ReactDOM的对象，其中包含了它暴露的API

```js
var ReactDOM = {
  findDOMNode: findDOMNode,
  render: ReactMount.render,
  unmountComponentAtNode: ReactMount.unmountComponentAtNode,
  version: ReactVersion,

  /* eslint-disable camelcase */
  unstable_batchedUpdates: ReactGenericBatching.batchedUpdates,
  unstable_renderSubtreeIntoContainer: ReactMount.renderSubtreeIntoContainer,
  /* eslint-enable camelcase */

  __SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED: {
    // For TapEventPlugin which is popular in open source
    EventPluginHub: require('EventPluginHub'),
    // Used by test-utils
    EventPluginRegistry: require('EventPluginRegistry'),
    EventPropagators: require('EventPropagators'),
    ReactControlledComponent: require('ReactControlledComponent'),
    ReactDOMComponentTree,
    ReactDOMEventListener: require('ReactDOMEventListener'),
    ReactUpdates: ReactUpdates,
  },
};
```



