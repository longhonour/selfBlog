# React router v4 在组件外通过JS跳转

>React router升级到v4版本后，不再提供导出browerHistory的功能。如果想要在代码中跳转，只能通过组件里的props中所包含的history操作，无法完成在组件外通过JS代码跳转的功能。如果想要自己封装一些服务，比如说为AJAX请求添加拦截器的功能，不通过拦截器的请求强制跳转到登陆页面，这种操作就无法实现了。

其实React router已经给出了解决办法，这里要用到一个额外的库 [history](https://github.com/ReactTraining/history "history") 具体用法如下：

> 安装

可以通过npm包管理器安装 `npm i history --save` 或者通过yarn包管理器安装 `yarn add history`

>引入

可以通过CommonJS的方式引入
```javascript
const createBrowserHistory = require('history').createBrowserHistory;
```
或者通过ES6标准的写法引入
```javascript
import { createBrowserHistory } from 'history';
```
也可以通过<script>标签引入
```javascript
<script src="history.min.js"></script>
```

>使用

首先创建history对象，通过代码：
```javascript
const history = createBrowserHistory();
```
>这里拿到的history对象和在组件中拿到的 this.props.history 的API都是一样的，可以直接使用，不过如果直接写作history.push('/index')则会只有地址栏的URL变化了，路由却没有切换。在react-router v4版本中，官方推荐使用<BrowserRouter>作为路由，但是由于<BrowserRouter>没有额外兼容这个库，所以我使用<BrowserRouter>作为路由后仍在外部使用代码控制跳转依然失效。正确的用法应该是这样：

```javascript
import React, { Component } from "react";
import { BrowserRouter, Router, Route } from "react-router-dom";
import { createBrowserHistory } from 'history';
...

const history = createBrowserHistory();
class App extends Component {
    render() {
        return (
		// 外部控制的<BrowserRouter>替换为<Router>，并将创建的history对象作为props传递给<Router>组件
            <Router history={history}>
                <div className="body">
                    <Route path="/login" component={Login} />
                    <Route path="/main" render={props => {
                        return (
						//BrowserRouter依旧有效
                            <BrowserRouter basename="/main">
                                <div className="App">
                                    <NavBar />
                                    <SideBar />
                                    <MainPage />
                                </div>
                            </BrowserRouter>
                        );
                    }} />
                </div>
            </Router>
        )
    }
}

history.push('/login');

```

>将你需要在外部控制的<BrowserRouter>替换为<Router>，并将创建的history对象作为props传递给<Router>组件，这样在外部通过操作history对象就可以正确跳转了。

>当然，也可以把history单独封装为一个模块，方便其他地方使用：

```javascript
/* history.jsx */

import { createBrowserHistory } from 'history';

const history = createBrowserHistory();
export default history;
```
