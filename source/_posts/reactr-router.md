## 路由原理

利用浏览器自带的路由 history api 二次封装实现 react router

- hashHistory，通过监听 hashChange 变化
- browserHistory，通过监听 popstate 的变化，浏览器 没有 pushState，但是可以通过自定义事件进行 aop 重写方法

```javascript
// aop 重写模拟 pushState
window.onpushstate = (state, pathname) => {
  console.log(state, pathname);
};
const historyObj = window.history;
const oldPushState = historyObj.pushState;
historyObj.pushState = (state, title, pathname) => {
  let result = oldPushState.call(historyObj, state, title, pathname);
  if (typeof window.onpushstate === "function") {
    // window.onpushstate(new CustomEvent('pushstate',{detail:{pathname,state}})); // 也可以自定义事件
    window.onpushstate(state, pathname);
  }
  return result;
};
```

```javascript
function createBrowserHistory() {
  const globalHistory = window.history;
  let listeners = []; //存放所有的监听函数
  let state;
  let message;
  function listen(listener) {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter((item) => item !== listener);
    };
  }
  function go(n) {
    globalHistory.go(n);
  }
  window.addEventListener("popstate", () => {
    //TODO
    let location = {
      state: globalHistory.state,
      pathname: window.location.pathname,
    };
    //当路径改变之后应该让history的监听函数执行，重新刷新组件
    notify({ action: "POP", location });
  });
  function goBack() {
    go(-1);
  }
  function goForward() {
    go(1);
  }
  function notify(newState) {
    //把newState上的属性赋值到history对象上
    Object.assign(history, newState);
    history.length = globalHistory.length; //路由历史栈中历史条目的长度
    listeners.forEach((listener) => listener(history.location)); //通知监听函数执行,参数是新的location
  }
  function push(pathname, nextState) {
    //TODO
    const action = "PUSH"; //action表示是由于什么样的动作引起了路径的变更
    if (typeof pathname === "object") {
      state = pathname.state;
      pathname = pathname.pathname;
    } else {
      state = nextState; //TODO
    }
    if (message) {
      let confirmMessage = message({ pathname });
      let allow = window.confirm(confirmMessage);
      if (!allow) return;
    }
    globalHistory.pushState(state, null, pathname); //我们已经 跳转路径
    let location = { state, pathname };
    notify({ action, location });
  }
  function block(newMessage) {
    message = newMessage;
    return () => (message = null);
  }
  const history = {
    action: "POP",
    go,
    goBack,
    goForward,
    push,
    listen,
    block,
    location: {
      pathname: window.location.pathname,
      state: window.location.state,
    },
  };
  return history;
}
export default createBrowserHistory;
```

```javascript
/**
 * hash不能使用 浏览器的history对象了
 * @returns
 */
function createHashHistory() {
  let stack = []; //类似于历史栈 里面存放都是路径
  let index = -1; //栈的指针，默认是-1
  let action = "POP"; //动作
  let state; //最新的状态
  let listeners = []; //监听函数的数组
  function listen(listener) {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter((item) => item !== listener);
    };
  }
  function go(n) {
    action = "POP";
    index += n; //更改栈顶的指针
    let nextLocation = stack[index]; //取出指定索引对应的路径对象
    state = nextLocation.state; //取出此location对应的状态
    window.location.hash = nextLocation.pathname; //修改hash值 ，从而修改当前的路径
  }
  let hashChangeHandler = () => {
    let pathname = window.location.hash.slice(1); //取出最新的hash值对应的路径  #/user
    Object.assign(history, { action, location: { pathname, state } });
    if (action === "PUSH") {
      //说明是调用push方法，需要往历史栈中添加新的条目
      stack[++index] = history.location;
    }
    listeners.forEach((listener) => listener(history.location));
  };
  function push(pathname, nextState) {
    action = "PUSH";
    if (typeof pathname === "object") {
      state = pathname.state;
      pathname = pathname.pathname;
    } else {
      state = nextState;
    }
    window.location.hash = pathname;
  }
  //当hash发生变化的话，会执行回调
  window.addEventListener("hashchange", hashChangeHandler);
  function goBack() {
    go(-1);
  }
  function goForward() {
    go(1);
  }
  const history = {
    action: "POP",
    go,
    goBack,
    goForward,
    push,
    listen,
    location: { pathname: "/", state: undefined },
  };
  if (window.location.hash) {
    //如果初始的情况下，如果hash是有值的
    action = "PUSH";
    hashChangeHandler();
  } else {
    window.location.hash = "/";
  }
  return history;
}
export default createHashHistory;
```

```javascript
// RouterContext
import React from "react";
export default React.createContext({});
```

```javascript
import React from "react";
import { Router } from "../react-router";
import { createHashHistory } from "../history";
class HashRouter extends React.Component {
  history = createHashHistory(); //HashRouter的history实例属性会指向用hash实现的历史对象
  render() {
    return <Router history={this.history}>{this.props.children}</Router>;
  }
}
export default HashRouter;
/**
 * createHashHistory和createBrowserHistory
 * 都 会反回一个history对象，对象的方法和API是完全 相同 的，只是内闻的实现原理不一样
 */
```

```javascript
import React from "react";
import { Router } from "../react-router";
import { createBrowserHistory } from "../history";
class BrowserRouter extends React.Component {
  history = createBrowserHistory(); //HashRouter的history实例属性会指向用hash实现的历史对象
  render() {
    return <Router history={this.history}>{this.props.children}</Router>;
  }
}
export default BrowserRouter;
```

```javascript
import React from "react";
import RouterContext from "./RouterContext";
class Router extends React.Component {
  static computeRootMatch(pathname) {
    return { path: "/", url: "/", params: {}, isExact: pathname === "/" };
  }
  constructor(props) {
    super(props);
    this.state = {
      location: props.history.location,
    };
    //监听历史对象路径变化，如果路径发生变化的话执行回调
    this.unlisten = props.history.listen((location) => {
      this.setState({ location });
    });
  }
  componentWillUnmount() {
    this.unlisten && this.unlisten();
  }
  render() {
    let value = {
      history: this.props.history,
      location: this.state.location,
      match: Router.computeRootMatch(this.state.location.pathname),
    };
    return (
      <RouterContext.Provider value={value}>
        {this.props.children}
      </RouterContext.Provider>
    );
  }
}
export default Router;
```

可以看到，通过二次封装 history api 然后经过 Provider 传递给组件，当路由变化的时候会调用 listen 方法，从而 `this.setState({ location })`, setState 则会更新视图，从而达到了 路由变化渲染不同视图的目的

## Route

```javascript
import pathToRegexp from "path-to-regexp"; // 匹配路径的全靠这个包

function compilePath(path, options) {
  const keys = [];
  const regexp = pathToRegexp(path, keys, options);
  return { regexp, keys };
}
function matchPath(pathname, options = {}) {
  const {
    path = "/",
    exact = false,
    strict = false,
    sensitive = false,
  } = options;
  const { regexp, keys } = compilePath(path, { end: exact, strict, sensitive });
  const match = regexp.exec(pathname);
  if (!match) return null;
  const [url, ...values] = match;
  const isExact = pathname === url; //
  if (exact && !isExact) return null; //如果希望精确，但其实不精确返回确
  return {
    path, //Route里的path属性
    url, //正则匹配到的浏览器的pathname部分
    isExact, //是否实现了精确匹配
    params: keys.reduce((memo, key, index) => {
      memo[key.name] = values[index];
      return memo;
    }, {}),
  };
}
/**
 * 浏览器的pathname  /user/1
 * path /user
 * match是能匹配上的
 * exact=true;
 * /user/1 不完全 相等/user  表示非精确匹配
 *
 *
 *
 *
 * Home   path = /
 * location.pathname /user
 * 匹配的部分就是 /
 * / === /user不相等就是false
 */
export default matchPath;
```

```javascript
import React from "react";
import RouterContext from "./RouterContext";
import matchPath from "./matchPath";
class Route extends React.Component {
  static contextType = RouterContext;
  render() {
    const { history, location } = this.context;
    const {
      component: RouteComponent,
      computedMatch,
      render,
      children,
    } = this.props;
    const match = computedMatch
      ? computedMatch
      : matchPath(location.pathname, this.props);
    const routeProps = { history, location };
    let renderElement = null; // null也一个合法的react渲染节点 代表我们render的返顺值，代表此组件将要渲染的内容
    if (match) {
      routeProps.match = match;
      //RouteComponent>render>children
      if (RouteComponent) {
        //如果传递了 component属性，优先渲染component
        renderElement = <RouteComponent {...routeProps} />;
      } else if (render) {
        renderElement = render(routeProps);
      } else if (children) {
        renderElement = children(routeProps);
      } else {
        renderElement = null;
      }
    } else {
      //TODO
      if (children) {
        renderElement = children(routeProps);
      } else {
        renderElement = null;
      }
    }
    return renderElement;
  }
}
export default Route;

/**
 * 指定一个route组件如何渲染有三种方式
 * 1.component 如果你渲染的是一个固定 的组件，确定的组件的话就可以component
 * 2.render 如果你想自己确认，自定义渲染逻辑就可以用render
 *
 * 1和2都是要求路径匹配才渲染或执行，如果路径不匹配什么不渲染
 * 3.children
 * 不管路由是否匹配，都渲染
 */
```

我们先看用法

```javascript
ReactDOM.render(
  <Router>
    <ul>
      <li>
        <Link to="/">首页</Link>
      </li>
      <li>
        <Link
          to={{ pathname: `/user/detail/1`, state: { id: 1, name: "张三" } }}
        >
          张三
        </Link>
      </li>
      <li>
        <Link to={{ pathname: `/post/1` }}>文章</Link>
      </li>
    </ul>
    <Route path="/" component={Home} />
    <Route path="/user/detail/:id" component={UserDetail} />
    <Route path="/post/:id" component={Post} />
  </Router>,
  document.getElementById("root")
);
```

- Router 组件 仅仅是 容器 把拿到的 路由信息 location 之类的
- Route 组件，就是根据 路由接收的 props 以及 我们 route 定义的 path 和 component 匹配，把匹配成功的 component 给渲染出来

## Switch 组件

```javascript
import React from "react";
import RouterContext from "./RouterContext";
import matchPath from "./matchPath";

class Switch extends React.Component {
  static contextType = RouterContext;
  render() {
    const { context } = this;
    const { children } = this.props;
    const { location } = context;
    let element, match;
    React.Children.forEach(children, (child) => {
      //child $$typeof == Symbol('react.element')
      if (React.isValidElement(child)) {
        //如果此节点是一个React元素
        if (!match) {
          //如果尚未有任何元素匹配
          element = child;
          match = matchPath(location.pathname, child.props);
        }
      }
    });
    return match ? React.cloneElement(element, { computedMatch: match }) : null;
  }
}

/* 
children=[Route,Route,Route]
React.Children.forEach = function(children,callback){
    let array = Array.isArray(children)?children:[children]
    array.filter(Boolean).forEach(callback);
} */
export default Switch;
```

- 不用 Switch 包裹 Route 组件，会把匹配到的 Route 组件全部渲染出来
- Switch 就是 优先返回匹配成功的 Route 组件

## Link 组件

```javascript
import React from "react";
import { __RouterContext as RouterContext } from "../react-router"; // 还是上面的 那个 RouterContext
export default function Link(props) {
  return (
    <RouterContext.Consumer>
      {(value) => {
        return (
          <a
            {...props}
            onClick={(event) => {
              event.preventDefault();
              value.history.push(props.to);
            }}
          >
            {props.children}
          </a>
        );
      }}
    </RouterContext.Consumer>
  );
}
```

- 很简单， 就是接收 history api 提供跳转功能

## Redirect

```javascript
import React from "react";

class Lifecycle extends React.Component {
  componentDidMount() {
    if (this.props.onMount) this.props.onMount(this);
  }
  componentWillUnmount() {
    if (this.props.onUnMount) this.props.onUnMount(this);
  }
  render() {
    return null;
  }
}
export default Lifecycle;
```

```javascript
import React from "react";
import RouterContext from "./RouterContext";
import Lifecycle from "./Lifecycle";

function Redirect({ to }) {
  return (
    <RouterContext.Consumer>
      {(value) => {
        const { history } = value;
        /*  history.push(to);
          return null; */
        return <Lifecycle onMount={() => history.push(to)} />;
      }}
    </RouterContext.Consumer>
  );
}

export default Redirect;
```

- 怕出现问题，用了个 空 Lifecycle 去跳转

## Protected

```javascript
import React from "react";
import { Route, Redirect } from "../react-router-dom";
interface Props extends Record<string, any> {
  path: string;
  component: React.ComponentType<any>;
}
export default (props: Props) => {
  let { component: RouteComponent, path } = props;
  return (
    <Route
      path={path}
      render={(props: any) =>
        localStorage.getItem("logined") ? (
          <RouteComponent {...props} />
        ) : (
          <Redirect
            to={{
              pathname: "/login",
              state: { from: props.location.pathname },
            }}
          />
        )
      }
    />
  );
};
```

- Protected 一般用于自定义路由，鉴权保护之类的，采用的是 renderProps 方式

## NavLink

```javascript
import React from "react";
import { Route, Link } from "./";
export default function NavLink(props) {
  const {
    to: path, //点击时要跳转换路径
    className: classNameprop = "",
    style: styleProp = {},
    activeClassName = "active", //激活类名
    activeStyle = {}, //激活行内样式
    children, //子节点
    exact, //是否精确匹配
  } = props;
  return (
    <Route path={path} exact={exact}>
      {({ match }) => {
        let className = match
          ? joinClassName(classNameprop, activeClassName)
          : classNameprop;
        let style = match ? { ...styleProp, ...activeStyle } : styleProp;
        let linkProps = { className, style, to: path, children };
        return <Link {...linkProps} />;
      }}
    </Route>
  );
}
// joinClassName(basic,active)=> "basic active"
function joinClassName(...classNames) {
  return classNames.filter((c) => c).join(" ");
}
```

- NavLink 一般用于展示激活态的样式，放在 Route 组件里的 children 渲染，无论是否匹配都会渲染

## withRouter

```javascript
import React from "react";
import RouterContext from "./RouterContext";
//高阶组件React中最重要的设计模式，没有之一
function withRouter(OldComponent) {
  function NewComponent(props) {
    return (
      <RouterContext.Consumer>
        {(value) => {
          return <OldComponent {...value} {...props} />;
        }}
      </RouterContext.Consumer>
    );
  }
  return NewComponent;
}
export default withRouter;
/* import {Route} from './';
//高阶组件 属性代理
export default function withRouter(OldComponent) {
    //TODO
    return (
        (props)=><Route render={
            routeProps=><OldComponent {...routeProps} {...props}/>
        }/>
    )
}
 */
```

- withRouter 组件，一般用于包裹那些 没有 路由跳转能力的 组件，提供 history api， 采用 renderProps 或者 高阶组件的方式去 透传

## Prompt

```javascript
import React, { Component } from "react";
import Lifecycle from "./Lifecycle"; //
import RouterContext from "./RouterContext";
function Prompt({ when, message }) {
  let value = React.useContext(RouterContext);
  React.useEffect(() => {
    if (when) return value.history.block(message);
  });
  return null;
}

// 源码 还是 用 Lifecycle
// function Prompt({ when, message }) {
//   return (
//     <RouterContext.Consumer>
//       {(value) => {
//         if (!when) return null;
//         const block = value.history.block;
//         return (
//           <Lifecycle
//             onMount={(inst) => (inst.release = block(message))}
//             onUnMount={(inst) => inst.release()}
//           />
//         );
//       }}
//     </RouterContext.Consumer>
//   );
// }
export default Prompt;
```

```javascript
let message
function push(pathname, nextState) {
  // ... more code
  if (message) {
    let confirmMessage = message({ pathname });
    let allow = window.confirm(confirmMessage);
    if (!allow) return;
  }
  globalHistory.pushState(state, null, pathname); //我们已经 跳转路径
  let location = { state, pathname };
  notify({ action, location });
}
function block(newMessage) {
  message = newMessage;
  return () => (message = null);
}

<Prompt
  when={true}
  message={(location) => `是否离开当前路径${location.pathname}`}
>
```

- Prompt 组件， 可以看到，window.confirm 返回 false 就走不下去了，也就不会跳转页面， 一般是离开页面挽留的作用

## hook

```javascript
import React from "react";
import RouterContext from "./RouterContext";
import matchPath from "./matchPath";

export function useParams() {
  let match = React.useContext(RouterContext).match;
  return match ? match.params : {};
}
export function useHistory() {
  return React.useContext(RouterContext).history;
}

export function useLocation() {
  return React.useContext(RouterContext).location;
}

export function useRouteMatch(options) {
  const location = useLocation();
  return matchPath(location.pathname, options);
}
```

- hooks 很简单，就是直接获取 history api

## 总结

通过 hashChange 和 popState 变化，二次封装 history api， 然后通过 provider 传递 给 Router，Rouer 作为 容器 渲染 Route 组件， Route 组件把接收的 provider props 和 自身 传入 的 props 匹配，以此来渲染 路由 对应的 组件
