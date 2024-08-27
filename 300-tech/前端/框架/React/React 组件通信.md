---
title: React 组件通信
date: 2024-08-22T01:51:33+08:00
updated: 2024-08-22T01:51:54+08:00
dg-publish: false
source: https://github.com/peng-yin/note/issues/16
---

## 父组件内调用子组件方法

```
&lt;Loading ref={el =&gt; {this.el = el}} /&gt;

this.el.changeLoading(false); // 子组件某个方法

```

## props 解决数据共享

通过 props 解决数据共享问题，本质上是将数据获取的逻辑放到组件的公共父组件中。代码可能是这样的：

```
import React, { PureComponent } from 'react';

export default class App extends PureComponent {
 constructor(props) {
    super(props);
    this.state = {
      data: {
        sku: '',
        desc: '',
      },
    };
  }

  componentDidMount() {
    fetch('url', { id: this.props.id })
      .then(resp =&gt; resp.json())
      .then(data =&gt; this.setState({ data }));
  }

  render() {
    return (
      &lt;div&gt;
        &lt;ProductInfoOne data={this.state.data} /&gt;
        &lt;ProductInfoTwo data={this.state.data} /&gt;
      &lt;/div&gt;
    );
  }
}

function ProductInfoOne({ data }) {
  const { sku } = data;
  return &lt;div&gt;{sku}&lt;/div&gt;;
}

function ProductInfoTwo({ data }) {
  const { desc } = data;
  return &lt;div&gt;{desc}&lt;/div&gt;;
}
```

问题是随着嵌套的层次越来越深，数据需要从最外层一直传递到最里层，整个代码的可读性和维护性会变差。我们希望打破数据「层层传递」而子组件也能取到父辈组件中的数据。

## Context API

React 16.3 的版本引入了新的 Context API，Context API 本身就是为了解决嵌套层次比较深的场景中数据传递的问题，看起来非常适合解决我们上面提到的问题。我们尝试使用 Context API 来解决我们的问题：

```
// context.js
const ProductContext = React.createContext({
  sku: '',
  desc: '',
});

export default ProductContext;

// App.js
import React, { PureComponent } from 'react';
import ProductContext from './context';

const Provider = ProductContext.Provider;

export default class App extends PureComponent {
 constructor(props) {
    super(props);
    this.state = {
      data: {
        sku: '',
        desc: '',
      },
    };
  }

  componentDidMount() {
    fetch('url', { id: this.props.id })
      .then(resp =&gt; resp.json())
      .then(data =&gt; this.setState({ data }));
  }

  render() {
    return (
      &lt;Provider value={this.state.data}&gt;
        &lt;ProductInfoOne /&gt;
        &lt;ProductInfoTwo /&gt;
      &lt;/Provider&gt;
    );
  }
}

// ProductInfoOne.js
import React, { PureComponent } from 'react';
import ProductContext from './context';

export default class ProductInfoOne extends PureComponent {
  static contextType = ProductContext;

  render() {
    const { sku } = this.context;
    return &lt;div&gt;{sku}&lt;/div&gt;;
  }
}

// ProductInfoTwo.js
import React, { PureComponent } from 'react';
import ProductContext from './context';

export default class ProductInfoTwo extends PureComponent {
  static contextType = ProductContext;

  render() {
    const { desc } = this.context;
    return &lt;div&gt;{desc}&lt;/div&gt;;
  }
}
```

## Redux

[![](https://camo.githubusercontent.com/925052c95888d83af52b5827f7dd9300d29e9e8bdb92d01a880163a2012f8f83/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031392f312f32332f313638373863663538343937353732363f696d61676556696577322f302f772f313238302f682f3936302f666f726d61742f776562702f69676e6f72652d6572726f722f31)](https://camo.githubusercontent.com/925052c95888d83af52b5827f7dd9300d29e9e8bdb92d01a880163a2012f8f83/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031392f312f32332f313638373863663538343937353732363f696d61676556696577322f302f772f313238302f682f3936302f666f726d61742f776562702f69676e6f72652d6572726f722f31)

```
// store.js
import { createStore } from 'redux';
import reducer from './reducer';

const store = createStore(reducer);

export default store;

// reducer.js
import * as actions from './actions';
import { combineReducers } from 'redux';

function ProductInfo(state = {}, action) {
  switch (action.type) {
    case actions.SET_SKU: {
      return { ...state, sku: action.sku };
    }
    case actions.SET_DESC: {
      return { ...state, desc: action.desc };
    }
    case actions.SET_DATA: {
      return { ...state, ...action.data };
    }
    default: {
      return state;
    }
  }
}

const reducer = combineReducers({
  ProductInfo,
});

export default reducer;

// action.js
export const SET_SKU = 'SET_SKU';
export const SET_DESC = 'SET_DESC';
export const SET_DATA = 'SET_DATA';

export function setSku(sku) {
  return {
    type: SET_SKU,
    sku,
  };
}

export function setDesc(desc) {
  return {
    type: SET_DESC,
    desc,
  };
}

export function setData(data) {
  return {
    type: SET_DESC,
    data,
  };
}

// App.js
import React, { PureComponent } from 'react';
import { Provider } from 'react-redux';
import store from './store';
import * as actions from './actions';

class App extends PureComponent {
  componentDidMount() {
    fetch('url', { id: this.props.id })
      .then(resp =&gt; resp.json())
      .then(data =&gt; this.props.dispatch(actions.setData(data)));
  }

  render() {
    return (
      &lt;Provider store={store}&gt;
        &lt;ProductInfoOne /&gt;
        &lt;ProductInfoTwo /&gt;
      &lt;/Provider&gt;
    );
  }
}

function mapStateToProps() {
  return {

  };
}

function mapDispatchToProps(dispatch) {
  return {
    dispatch,
  };
}

export default connect(mapStateToProps, mapDispatchToProps)(App);

// ProductInfoOne.js
import React, { PureComponent } from 'react';
import { connect } from 'react-redux';
import * as actions from './actions';

class ProductInfoOne extends PureComponent {
  onEditSku = (sku) =&gt; {
    this.props.dispatch(actions.setSku(sku));
  };

  render() {
    const { sku } = this.props.data;
    return (
      &lt;div&gt;{sku}&lt;/div&gt;
    );
  }
}

function mapStateToProps(state) {
  return {
    data: state.ProductInfo,
  };
}

function mapDispatchToProps(dispatch) {
  return {
    dispatch,
  };
}

export default connect(mapStateToProps, mapDispatchToProps)(ProductInfoOne);

// ProductInfoTwo.js
import React, { PureComponent } from 'react';
import { connect } from 'react-redux';
import * as actions from './actions';

class ProductInfoTwo extends PureComponent {
  onEditDesc = (desc) =&gt; {
    this.props.dispatch(actions.setDesc(desc));
  };

  render() {
    const { desc } = this.props.data;
    return (
      &lt;div&gt;{desc}&lt;/div&gt;
    );
  }
}

function mapStateToProps(state) {
  return {
    data: state.ProductInfo,
  };
}

function mapDispatchToProps(dispatch) {
  return {
    dispatch,
  };
}

export default connect(mapStateToProps, mapDispatchToProps)(ProductInfoTwo);

```

## Mobx

Mobx 我理解的最大的好处是简单、直接，数据发生变化，那么界面就重新渲染，在 React 中使用时，我们甚至不需要关注 React 中的 state，我们看下用 MobX 怎么解决我们上面的问题

[![mobx](https://camo.githubusercontent.com/5ed3eeb0297f489a2d9d69390727c1f256f56bb000e58926f4839a0094a65f39/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031392f312f32332f313638373863663538336463653930623f696d61676556696577322f302f772f313238302f682f3936302f666f726d61742f776562702f69676e6f72652d6572726f722f31)](https://camo.githubusercontent.com/5ed3eeb0297f489a2d9d69390727c1f256f56bb000e58926f4839a0094a65f39/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031392f312f32332f313638373863663538336463653930623f696d61676556696577322f302f772f313238302f682f3936302f666f726d61742f776562702f69676e6f72652d6572726f722f31)

```
// store.js
import { observable } from 'mobx';

const store = observable({
  sku: '',
  desc: '',
});
export default store;

// App.js
import React, { PureComponent } from 'react';
import store from './store.js';

export default class App extends PureComponent {
  componentDidMount() {
    fetch('url', { id: this.props.id })
      .then(resp =&gt; resp.json())
      .then(data =&gt; Object.assign(store, data));
  }

  render() {
    return (
      &lt;div&gt;
        &lt;ProductInfoOne /&gt;
        &lt;ProductInfoTwo /&gt;
      &lt;/div&gt;
    );
  }
}

// ProductInfoOne.js
import React, { PureComponent } from 'react';
import { action } from 'mobx';
import { observer } from 'mobx-react';
import store from './store';

@observer
class ProductInfoOne extends PureComponent {
  @action
  onEditSku = (sku) =&gt; {
    store.sku = sku;
  };

  render() {
    const { sku } = store;
    return (
      &lt;div&gt;{sku}&lt;/div&gt;
    );
  }
}

export default ProductInfoOne;

// ProductInfoTwo.js
import React, { PureComponent } from 'react';
import { action } from 'mobx';
import { observer } from 'mobx-react';
import store from './store';

@observer
class ProductInfoTwo extends PureComponent {
  @action
  onEditDesc = (desc) =&gt; {
    store.desc = desc;
  };

  render() {
    const { desc } = store;
    return (
      &lt;div&gt;{desc}&lt;/div&gt;
    );
  }
}

export default ProductInfoTwo;

```

## GraphQL
