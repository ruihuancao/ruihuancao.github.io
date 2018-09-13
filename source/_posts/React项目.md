---
title: React开发
date: 2016-04-12 20:10:13
categories:
- React
tags:
- React
---

React 是一个用于构建用户界面的 JAVASCRIPT 库。在研究React Native的时候顺便看了一下，涉及到很多前端的知识，很多都没有深入了解，等以后完善。

<!--more-->

## React 项目搭建
需要先安装node，自行安装
### 新建目录

```
mkdir reactTemplate
cd reactTemplate
npm init
touch webpack.config.js
touch server.js
touch index.html
touch .gitgnore
touch .babelrc

mkdir dist

mkdir src
cd src
mkdir store
mkdir reducers
mkdir containers
mkdir components
mkdir actions
```

### webpack.config.js
```
var path = require('path')
var webpack = require('webpack')

module.exports = {
  devtool: 'source-map',
  entry: [
    // 'webpack-hot-middleware/client',
    './index'
  ],
  output: {
    path: path.join(__dirname, 'static'),
    //path: path.join("../", 'static'),
    filename: 'bundle.js',
    publicPath: '/static/'
  },
  plugins: [
    new webpack.optimize.OccurrenceOrderPlugin(),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoErrorsPlugin()
  ],
  module: {
    loaders: [
      {
        test: /\.js$/,
        loaders: [ 'babel' ],
        exclude: /node_modules/,
        include: __dirname
      }
    ]
  }
}
```

### server.js
```
var webpack = require('webpack')
var webpackDevMiddleware = require('webpack-dev-middleware')
var webpackHotMiddleware = require('webpack-hot-middleware')
var config = require('./webpack.config')

var app = new (require('express'))()
var port = 3000

var compiler = webpack(config)
app.use(webpackDevMiddleware(compiler, { noInfo: true, publicPath: config.output.publicPath }))
app.use(webpackHotMiddleware(compiler))

app.use(function(req, res) {
  res.sendFile(__dirname + '/index.html')
})

app.listen(port, function(error) {
  if (error) {
    console.error(error)
  } else {
    console.info("==> 🌎  Listening on port %s. Open up http://localhost:%s/ in your browser.", port, port)
  }
})
```

### .gitgnore
```
node_modules
.idea/
```
### .babelrc
```
{
  "presets": ["es2015", "react"],
  "env": {
    "development": {
      "presets": ["react-hmre"]
    }
  }
}
```
### index.js
```
import 'babel-polyfill'
import React from 'react'
import { render } from 'react-dom'
import Root from './containers/Root'
import configureStore from './store/configureStore'

const store = configureStore()

render(
  <Root store={store} />,
  document.getElementById('root')
)
```
### index.html
```
<!doctype html>
<html class="no-js" lang="">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Material-UI Example</title>
    <meta name="description" content="Google's material design UI components built with React.">

    <!-- Use minimum-scale=1 to enable GPU rasterization -->
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1, user-scalable=0, maximum-scale=1, minimum-scale=1"
    >
    <!-- <link rel="stylesheet" type="text/css" href="main.css"> -->

    <style type="text/css">
    html {
      font-family: 'Roboto', sans-serif;
    }
    body {
      font-size: 13px;
      line-height: 20px;
      padding: 0px;
      margin: 0px;
    }
   </style>
  </head>

  <body>
    <div id="root">
    </div>

    <!-- This script adds the Roboto font to our project. For more detail go to this site:  http://www.google.com/fonts#UsePlace:use/Collection:Roboto:400,300,500 -->
    <script>
      var WebFontConfig = {
        google: { families: [ 'Roboto:400,300,500:latin' ] }
      };
      (function() {
        var wf = document.createElement('script');
        wf.src = 'https://ajax.googleapis.com/ajax/libs/webfont/1/webfont.js';
        wf.type = 'text/javascript';
        wf.async = 'true';
        var s = document.getElementsByTagName('script')[0];
        s.parentNode.insertBefore(wf, s);
      })();
    </script>
    <script src="/static/bundle.js"></script>
  </body>
</html>
```
### package.json
```
{
  "name": "react",
  "version": "1.0.0",
  "description": "react page",
  "main": "index.js",
  "scripts": {
    "start": "node server.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "babel-polyfill": "^6.3.14",
    "isomorphic-fetch": "^2.1.1",
    "material-ui": "^0.16.4",
    "react": "^15.4.1",
    "react-dom": "^15.4.1",
    "react-redux": "^4.2.1",
    "react-router": "^2.0.0",
    "react-router-redux": "^4.0.0",
    "react-tap-event-plugin": "^2.0.1",
    "redux": "^3.2.1",
    "redux-logger": "^2.4.0",
    "redux-thunk": "^1.0.3"
  },
  "devDependencies": {
    "babel-core": "^6.3.15",
    "babel-loader": "^6.2.0",
    "babel-preset-es2015": "^6.3.13",
    "babel-preset-react": "^6.3.13",
    "babel-preset-react-hmre": "^1.1.1",
    "expect": "^1.6.0",
    "express": "^4.13.3",
    "node-libs-browser": "^0.5.2",
    "webpack": "^1.9.11",
    "webpack-dev-middleware": "^1.2.0",
    "webpack-hot-middleware": "^2.9.1",
    "redux-devtools": "^3.3.1",
    "redux-devtools-dock-monitor": "^1.1.1",
    "redux-devtools-log-monitor": "^1.0.11"
  }
}
```

### configureStore.js
```
import { createStore, applyMiddleware, compose } from 'redux'
import thunkMiddleware from 'redux-thunk'
import createLogger from 'redux-logger'
import rootReducer from '../reducers'
import DevTools from '../containers/DevTools'

export default function configureStore(preloadedState) {
  const store = createStore(
    rootReducer,
    preloadedState,
    compose(
      applyMiddleware(thunkMiddleware, createLogger()),
      DevTools.instrument()
    )
  )
  if (module.hot) {
    module.hot.accept('../reducers', () => {
      const nextRootReducer = require('../reducers')
      store.replaceReducer(nextRootReducer)
    })
  }
  return store
}
```
### action
```
import fetch from 'isomorphic-fetch'

export const TODOS_REQUEST = 'TODOS_REQUEST'
export const TODOS_SUCCESS = 'TODOS_SUCCESS'
export const TODOS_FAILURE = 'TODOS_FAILURE'

export function loadTodos() {
  console.log("获取数据");
  return (dispatch, getState) => {
    dispatch(requestTodos());
    return fetch('/todo/api/v1.0/todos')
      .then(response =>
        response.json().then(json => ({ json, response }))
      ).then(({ json, response }) => {
          if (!response.ok) {
              dispatch(requestTodosFailure())
          }
          dispatch(requestTodosSuccess(json))
      })
  }
}

export function requestTodoList() {
  dispatch(requestTodos());
  return fetch('/todo/api/v1.0/todos')
    .then(response =>
      response.json().then(json => ({ json, response }))
    ).then(({ json, response }) => {
        console.log(json);
        if (!response.ok) {
            dispatch(requestTodosFailure())
        }
        console.log(json);
        dispatch(requestTodosSuccess(json))
    })
}

function requestTodos() {
  return {
    type: TODOS_REQUEST,
  }
}

function requestTodosFailure() {
  return {
    type: TODOS_FAILURE,
  }
}

function requestTodosSuccess(json) {
  console.log(json);
  return {
    type: TODOS_SUCCESS,
    data: json,
    receivedAt: Date.now()
  }
}
```
### reducers
```
import * as ActionTypes from '../actions'

const selectedTodoList = (
    state = {
        isFetching: false
    },
        action) =>
{
  switch (action.type) {
    case ActionTypes.TODOS_REQUEST:
      return Object.assign({}, state, {
            isFetching: true
          })
    case ActionTypes.TODOS_SUCCESS:
          return  Object.assign({}, state, {
            isFetching: false,
            todos : action.data,
            lastUpdated: action.receivedAt
          })
    case ActionTypes.TODOS_FAILURE:
           return  Object.assign({}, state, {
            isFetching: false
          })
    default:
      return state
  }
}

export default selectedTodoList
```
### Root.js

```
import React, { Component, PropTypes } from 'react'
import { Provider } from 'react-redux'
import { Router, Route, IndexRoute, browserHistory } from 'react-router'
import { syncHistoryWithStore, routerReducer } from 'react-router-redux'

import DevTools from './DevTools'
import App from './App'
import Two from './Two'
import Three from './Three'
import Home from './Home'

export default class Root extends Component {
  render() {
    const { store } = this.props
    const history = syncHistoryWithStore(browserHistory, store)
    return (
      <Provider store={store}>
        <div>
          <Router history={history}>
            <Route path="/react" component={App}>
              <IndexRoute component={Home}/>
              <Route path="/react/two" component={Two}/>
              <Route path="/react/three" component={Three}/>
            </Route>
          </Router>
          <DevTools />
        </div>
      </Provider>
    )
  }
}

Root.propTypes = {
  store: PropTypes.object.isRequired
}
```

## React组件
组件继承自Component
constructor() 构造函数
componentDidMount() 声明周期中的一个方法，可以在此发起网络请求  
render 绘制完成执行,返回组件view
App.propTypes 设置组件参数
mapStateToProps 绑定参数，及Action到组件
connect（） 连接action 和reducers 到组件

##  redux 框架
用于方便复杂的管理state。
action描述了有事情发生了这一事实,
reducers 负责更新state,
store连接action和reducers 。
严格的单向数据流是 Redux 架构的设计核心。

action : 传递数据从应用到store. 操作网络。或者数据库，返回action type
reducers : 根据action type 更新state
store: 把action 和reducers联系到一起的对象。

createStore方法创建store reducers 作为参数传入 当需要操作时 store.dispatch(action) state: 全局状态树，维护组件数据。 与view组件绑定，当state变化，view刷新

单向数据流： UI操作-->调用store.dispatch(action)-->action做一些业务操作，返回type--> reducers根据type做相应处理，更新state--> 新的state更新UI

redux-thunk : 一个用于实现异步的中间件。 中间间三层调用 store => next => action =>

个人应用开发技术栈：
web端：
后端python，web框架Flask部署在leancoud，定时任务python爬虫抓取数据，提供restful api。
前端react，css框架bootstrap构建页面。
移动端： React Native。Android端目前并不是很成熟，不知何时能有稳定版本。
