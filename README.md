# 整体架构React-redux-chatapp

- [项目安装](#)
- [项目打包](#)
- [项目启动](#)
- [前期项目准备](#)
- [前后端端口不一致的解决方法](#)
- [antd-mobile插件](#antd-mobile)
- [redux基础准备](#redux)
- [手动链接react和redux](#reactredux)
- [使用react-redux插件来优雅的连接react和redux](#react-reduxreactredux)
- [使用connect装饰器](#connect)
- [App实现过程](#app)
  - [页面文件结构](#)
- [开发模式](#)
  - [登录注册页面](#)
    - [登录组件](#)
    - [注册组件](#)
    - [判断路由组件（AuthRoute）](#authroute)
      - [后端用户信息模拟）](#)
  - [完善信息（注册完之后）](#)
    - [BOSS页面](#boss)
      - [头像组件](#)
    - [牛人完善信息页面](#)
  - [页面Dashboard](#dashboard)
    - [Boss身份所看到的是牛人列表](#boss)
    - [牛人身份所看到的是Boss列表](#boss)
  - [高阶组件（HOC）](#hoc)
  - [Socket.io](#socketio)
    - [chat组件](#chat)
    - [Message消息列表的实现](#message)
    - [消息未读数量更新](#)
    - [收尾优化](#)
      - [性能优化](#)
      - [eslint代码校验工具](#eslint)
        - [服务器端渲染](#)
  - [React16 新特性](#react16)
- [Summary](#summary)
# 项目安装
```
npm install
```
# 项目打包
```
npm run build
```
# 项目启动
```
npm run server
```
# 前期项目准备
* Express+Mongodb
  Express开发Web接口，通过使用nodejs的mongoose模块链接和操作mongodb;即通过mongoose操作mongodb，mogodb存储的就是json数据，相对与Mysql要容易的的多
  mongoose类似于Mysql的形式，也有文档、字段的概念，也有常用的増删改查方法

```js
  const express = require('express');//引入express模块
  const mongoose = require('mongoose'); //引入mongoose模块
  const DB_URL = 'mongodb://localhost:27017/imooc';//链接mongo，启动mondo后的connection字段的值
  mongoose.connect(DB_URL);//连接
  //表示有两个字段
  const User = mongoose.model('user', new mongoose.Schema({
      user: {type: String, require: true},
      age: {type: Number, require: true}
  }))
  User.create({
     user: 'xuejiao',
     age: 25
     },function(err,doc){
     if(!err) {
                console.log(doc)
              }else{
         console.log(err)
     }
 })
  //把年龄18的删掉
  User.remove({age:18},function(err,doc){
    console.log(doc)
  })
  User.update({'user': 'xiaoming'},{'$set':{age:27}},function(err,doc){
     console.log(doc)
    })
```
* 为什么选择mogoDB这个数据库
1. 无数据结构的限制
    没有表结构的概念，每条记录可以有完全不同的结构;业务开发方便快捷;sql数据库需要事先定义表结构
2. 完全的索引支持
3. 方便扩展
4. 具有完善的文档支持
# 前后端端口不一致的解决方法
  1. 在webpack-dev-derver的proxy的属性中配置允许的路由接口
  2. 在package.ison文件中配置proxy;（本项目中使用）
  3. 还可以使用jsonp的形式，解决跨域的问题;
  * Jsonp的优点：它不像XMLHttpRequest对象实现的Ajax请求那样受到同源策略的限制;它的兼容性更好，在更加古老的浏览器中都可以运行，不需要XMLHttpRequest或ActiveX的支持;并且在请求完毕后可以通过调用callback的方式回传结果。
  * JSONP的缺点：他只支持GET请求而不支持POST等其他类型的HTTP请求，且只支持跨域HTTP请求这种情况，不能解决不同域的两个页面之间进行JavaScript调用的问题
  * json 和 jsonp这两个虽然只有一个字母的差别，但其实它们根本不是一回事;json是一种数据格式（即描述信息的格式），而jsonp是一种非官方的跨域数据交互协议（即信息传递双方约定的方法）;
  ** json的优点：
  1. 基于纯文本，跨平台传递及其简单;
  2. Javascript原生支持，后台语言几乎全部支持;
  3. 轻量级数据格式，占用字符数量极少，特别适合互联网传递;
  4. 可读性较强
  5. 容易编写和解析，但前提是你要知道数据结构
  json使用Javascript语法的子集表示对象，数组、布尔值、字符串、数值和null。

  * jsonp(一种非正式传输协议)，该协议的一个要点就是允许用户传递一个callback参数给服务端，然后服务端返回数据时会将这个callback参数作为函数名来包裹住json数据，这样客户端就可以随意定制自己的函数来自动处理返回的数据
  
# antd-mobile插件
基于react的一个UI库：[官网链接](https://mobile.ant.design/index-cn)
为了实现按需加载的功能，安装插件babel-plugin-import，并在package.json文件中进行配置
```json
"babel": {
    "plugins": [
      [
        "import",
        {
          "libraryName": "antd-mobile",
          "style": "css"
        }
      ],
    ]
}
```
# redux基础准备
1. 在搭配好的环境中，先手动实现redux，数据管理，粗糙代码如下：
```js
import { createStore } from 'redux';
//action.type
const HAND_IN_WEAPON = "HAND_IN_WEAPON";
const APPLY_FOR_WEAPON = "APPLY_FOR_WEAPON";

//reducer
function counter(state=10,action){
    switch(action.type) {
        case HAND_IN_WEAPON:
            return state-1
            break;
        case APPLY_FOR_WEAPON:
            return state+1
        default:
        return state
    }
}
//通过reducer建立store
const store = createStore(counter);
//利用store.getState()得到状态
const WeaponState=store.getState();
console.log(WeaponState)
//设置一个监听函数，监听state状态的变化
function render(){
    const WeaponState=store.getState()
    console.log(WeaponState)
}
//利用store.subscribe()函数订阅一下
store.subscribe(render);//注意，监听函数的位置
store.dispatch({type:HAND_IN_WEAPON});

store.dispatch({type:APPLY_FOR_WEAPON});

store.dispatch({type:APPLY_FOR_WEAPON});


```
* 上述一段代码主要是简单的阐述一下，就单单redux来管理自己的数据，大致过程是：
1. 通过reducer新建一个store，其中reducer是根据旧的状态和action生成新的state
2. 通过dispatch派发事件，传递action
3. 通过subscribe订阅事件，subscribe订阅render()函数，每次修改状态都会重新渲染

* 补充：如果全局属性较多，即对不同的全局属性有不同的reduce的时候，此时，需要一个combinReducer将所有的reducer合并起来，形成reducers，传给createrStore函数用于生成store。

* 总结一下redux工作的原理：首先，调用store.dispatch()将action作为参数传入，同时getState方法获取当前的状态树state并注册subscribe函数监听state的变化，再调用combineReducers并获取的state和action传入。combineReducer会将传入的state和action传给所有的reducer，然后reducer根据action.type返回新的state，触发state树的更新，我们调用subscribe监听到state变化后，用getState获取新的state数据

# 手动链接react和redux
1. 将action和reducer抽离出来，单独形成一个js文件，比如App.redux.js
```js
const HAND_IN_WEAPON = "HAND_IN_WEAPON";
const APPLY_FOR_WEAPON = "APPLY_FOR_WEAPON";

//reducer
export function counter(state=10,action){
    switch(action.type) {
        case HAND_IN_WEAPON:
            return state-1
            break;
        case APPLY_FOR_WEAPON:
            return state+1
        default:
        return state
    }
}


//action creator
export function handin(){
    return {type: HAND_IN_WEAPON}
}
export function apply(){
    return {type: APPLY_FOR_WEAPON}
}
```
2. 在入口文件中index.js（这个并不是本项目现在的这个入口文件，只是我举的一个例子）引入App.rudex.js，以及createStore;通过reducer以及createStore创建一个store，传入到子组件(App)中，让其作为props的一个属性值，关键代码如下：
```js
//index.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { createStore } from 'redux';
//reducers
import { counter ,handin,apply } from './weapon.redux';

const store = createStore(counter);


function render(){
    ReactDOM.render(
      <App store={store}/>,
      document.getElementById('root')
    );
}
render();
store.subscribe(render

//App.js
import React, { Component } from 'react';
// import logo from './logo.svg';
import {handin,apply } from './weapon.redux';


class App extends Component {
  constructor(props){
    super(props)
  }
  render() {
    const store = this.props.store;
    const num = store.getState();
    console.log(num);
    return (
      <div>
        <div>我们现在有武器{ num }把</div>
        <br/>
        <br/>
        <button onClick={()=>store.dispatch(handin())}>上交武器</button>
        <button onClick={()=>store.dispatch(apply())}>申请武器</button>
        <button>延迟上交武器</button>
      </div>
    );
  }
}
```
export default App;
# 使用react-redux插件来优雅的连接react和redux
至此，已经手动连接了react和redux;但是并不是很优雅，因此需要安装react-redux，该插件可优雅的链接react和redux;该插件提供两个比较好用的组件Provider和connect;
将上面的index.js和App.js这两个文件更新如下：
```js
// index.js

import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
//reducers
import { counter } from './weapon.redux';

const store = createStore(counter );

ReactDOM.render(
  (<Provider store={store}>
      <App />
  </Provider>),
  document.getElementById('root')
);

//App.js
import React, { Component } from 'react';
// import logo from './logo.svg';
import {handin,apply } from './weapon.redux';
import { connect } from 'react-redux'


class App extends Component {
  constructor(props){
    super(props)
  }
  render() {
    // console.log(this.props.num);
    return (
      <div>
        <div>我们现在有武器{ this.props.num }把</div>
        <br/>
        <br/>
        <button onClick={this.props.handin}>上交武器</button>
        <button onClick={this.props.apply}>申请武器</button>
        <button>延迟上交武器</button>
      </div>
    );
  }
}
function mapStateToProps(state){
  return {num: state}
}
const actionCreators = {handin,apply};
App=connect(
  mapStateToProps,
  actionCreators
)(App)
export default App;
```

* 在使用react-redux插件后，下面实现异步处理
比如说，实现一个1s或几秒过后在执行的一个action
由于异步处理action creator返回的是一个函数，并不是一个对象;但是createStore只能处理对象;因此，需要一个插件redux-thunk来实现异步处理，同时也需要一个中间件来对thunk进行开启;有了异步处理插件之后，Action就可以返回函数，并使用dispatch提交action，则上面的代码更新为：
```js
//index.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { createStore, applyMiddleware} from 'redux';//改动的部分
import { Provider } from 'react-redux';
import thunk from 'redux-thunk';//改动的部分
//reducers
import { counter } from './weapon.redux';


const store = createStore(counter,applyMiddleware(thunk));//改动的部分

ReactDOM.render(
  (<Provider store={store}>
      <App />
  </Provider>),
  document.getElementById('root')
);

//App.js

import React, { Component } from 'react';
// import logo from './logo.svg';
import {handin,apply,handinAsyn } from './weapon.redux';//改动的部分
import { connect } from 'react-redux'


class App extends Component {
  constructor(props){
    super(props)
  }
  render() {
    // console.log(this.props.num);
    return (
      <div>
        <div>我们现在有武器{ this.props.num }把</div>
        <br/>
        <br/>
        <button onClick={this.props.handin}>上交武器</button>
        <button onClick={this.props.apply}>申请武器</button>
        <button onClick={this.props.handinAsyn}>延迟上交武器</button>//改动的部分
      </div>
    );
  }
}
function mapStateToProps(state){
  return {num: state}
}
const actionCreators = {handin,apply,handinAsyn};//改动的部分
App=connect(
  mapStateToProps,
  actionCreators
)(App)
export default App;

//App.redux.js
const HAND_IN_WEAPON = "HAND_IN_WEAPON";
const APPLY_FOR_WEAPON = "APPLY_FOR_WEAPON";

//reducer
export function counter(state=10,action){
    switch(action.type) {
        case HAND_IN_WEAPON:
            return state-1
            break;
        case APPLY_FOR_WEAPON:
            return state+1
        default:
        return state
    }
}


//action creator
export function handin(){
    return {type: HAND_IN_WEAPON}
}
export function apply(){
    return {type: APPLY_FOR_WEAPON}
}
//改动的部分
export function handinAsyn(){
    return dispatch=>{
        setTimeout(()=>{
            dispatch(handin())
        },2000)
    }
}
```
* redux的调试工具Chrome中添加Redux DevTools
使用步骤：
1. 在新建store的时候判断window.devToolsExtension
2. 使用compose结合thunk和window.devToolsExtension(compose函数对多个函数进行组合)
3. 就可以在调试窗的redux选项卡里，实时看到stateconst

相应的index.js改为：
```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { createStore, applyMiddleware ,compose} from 'redux';
import { Provider } from 'react-redux';
import thunk from 'redux-thunk'
//reducers
import { counter } from './weapon.redux';

const reduxDevtools = window.devToolsExtension ? window.devToolsExtension() : f=>f
const store = createStore(counter,compose(
    applyMiddleware(thunk),
    reduxDevtools
    )
);

ReactDOM.render(
  (<Provider store={store}>
      <App />
  </Provider>),
  document.getElementById('root')
);

```
# 使用connect装饰器
1. 安装一个插件 babel-plugin-transform-decorators-legacy (开发的时候用);
2. 在package.json文件中进行配置
```json
"babel": {
    "presets": [
      "react-app"
    ],
    "plugins": ["transform-decorators-legacy"]//添加的部分
  },

  ```
  App.js更新为：
  ```js
//App.js
import React, { Component } from 'react';
// import logo from './logo.svg';
import {handin,apply,handinAsyn } from './weapon.redux';
import { connect } from 'react-redux';
@connect(
  //你要state什么属性放到props里
  state=>({num:state}),
  //你要什么方法，放到props里，自动dispatch
  {handin,apply,handinAsyn})
class App extends Component {
  constructor(props){
    super(props)
  }
  render() {
    // console.log(this.props.num);
    return (
      <div>
        <div>我们现在有武器{ this.props.num }把</div>
        <br/>
        <br/>
        <button onClick={this.props.handin}>上交武器</button>
        <button onClick={this.props.apply}>申请武器</button>
        <button onClick={this.props.handinAsyn}>延迟上交武器</button>
      </div>
    );
  }
}
// App=connect(
//   mapStateToProps,
//   actionCreators
// )(App)
export default App;
```



# App实现过程
## 页面文件结构
* 组件放在component文件夹下面
* 页面（业务组件）放在container文件夹下面
* 页面入口处获取用户信息，决定跳转到哪个页面
# 开发模式
* 基于cookie用户验证
express依赖cookie-parser（插件），cookie类似于一张身份卡，登录后服务器端返回，你带着cookie就可以访问受限资源（比如：牛人列表和Boss列表，需要有对应的token才能看到）
## 登录注册页面
  1. 首先，在入口文件中设置好相应的路由，以及对应的组件（登录，注册）
  ```js
  <Route path='/login' component={Login}></Route>
  <Route path='/register' component={Register}></Route>
  ```
  2. 然后分别实现两个基本组件，分别作为登录组件和注册组件，并且跑通;
  3. 因为登录和注册两个组件都有一个logo，因此将其抽离出来，实现为一个logo组件，使其可复用;
### 登录组件
  接着上面，在登录页面的logo下面实现登录输入和登录按钮;
  1. 从antd-mobile这个第三方库中，引入一些使用的组件，比如，List，InputItem，Button，WingBlank，WhiteSpace
  2. 用List包裹InputIntem以实现登录的输入的用户名和密码这两个输入框
  3. 在注册的Button上绑定跳转函数，使其可以跳转到注册页面;因为登录组件和注册组件是路由组件，因此可用histroy.push()实现页面的跳转
### 注册组件
1. 大部分和登录组件一样，只是多了一个身份选择，因此引入一个组件样式Radio，用其对应的标签RadioItem实现身份的包裹;
2. 在这里（目前），我们的身份信息是注册组件自己的内部状态，还不是从后端选取的;
3. 设定好注册页面的一些状态，这里有：用户名，密码，确认密码，身份这四个状态，然后在注册页面绑定相应的事件通过this.setState()修改状态以实现交互，这里使用的事件是onChange事件，因此在注册页面输入完成以后，可以把状态存储在state里面。
  ```js
import React from 'react';
import Logo from '../../component/logo/logo';
import {List, InputItem,Radio, WhiteSpace,Button} from 'antd-mobile';
class Register extends React.Component{
    constructor(props){
        super(props)
        this.state={
          user: '',
          pwd: '',
          repeatpwd: '',
          type: 'genius'
        }
        this.handleRegister=this.handleRegister.bind(this)
    }
    handleChange(key,val){
      this.setState({
        [key]:val   //注意这里一定要使用方括号
        })
    }
    handleRegister(){
        console.log(this.state)
    }
    render(){
        const RadioItem=Radio.RadioItem;
        return (
            <div>
            <Logo></Logo>
                <List>
                    <InputItem onChange={v=>this.handleChange('user',v)}> User Name </InputItem>
                    <WhiteSpace></WhiteSpace>
                    <InputItem type='password' onChange={v=>this.handleChange('pwd',v)}> Set password </InputItem>
                    <WhiteSpace ></WhiteSpace>
                    <InputItem type='password' onChange={v=>this.handleChange('repeatpwd',v)}> Confirm password </InputItem>
                    <WhiteSpace></WhiteSpace>
                    <RadioItem
                    checked={this.state.type==='genius'}
                    onChange={()=>this.handleChange('type','genius')}>Niu Ren</RadioItem>
                    <WhiteSpace></WhiteSpace>
                    <RadioItem
                    checked={this.state.type==='BOSS'}
                    onChange={()=>this.handleChange('type','boss')}>BOSS</RadioItem>
                    <Button type='primary' onClick={this.handleRegister}>REGUSTER</Button>
                </List>
            </div>)
    }
export default Register
```
4. 注册请求的发送：(发送给服务器)通过redux实现一个提交注册信息的功能
  * 通过redux管理数据;
  * action暂时有注册成功，注册失败两个action
    这里是否注册成功，以及注册失败都是通过获取后端的信息参数来决定的，因此格外需要一个action creator返回一个函数来处理异步请求
```js
import axios from 'axios';
const REGISTER_SUCCESS='REGISTER_SUCCESS';
const ERROR_MSG='ERROR_MSG';

const initState={
  isAuth:false,
  msg:'',
  user:'',
  pwd:'',
  type:''
}//初始状态
//创建了一个名叫user的reducer
export function user(state=initState,action){
    switch(action.type){
        case REGISTER_SUCCESS:
             return {...state,msg:'',isAuth: true,...action.payload}
        case ERROR_MSG:
            return {...state,isAuth:false,msg:action.msg}
        default:
            return state

    }
}

function registerSuccess(data){
  return {type:REGISTER_SUCCESS,payload: data}
}
function errorMsg(msg) {
  return {msg, type: ERROR_MSG }//注意这里msg其实是msg：msg的缩写，缩写后，需要放在最前面
}

export function register ({user,pwd,repeatpwd,type}){
  if(!user||!pwd||!type) {
    return errorMsg('用户名密码必须输入')
  }
  if(pwd!==repeatpwd) {
    return errorMsg('密码和确认密码不同')
  }
  return dispatch=>{
    axios.post('/user/register',{user,pwd,type})//向后端发送请求，将用户输入的数据全部传给后端
    .then(res=>{
      if(res.status==200 && res.data.code===0){
        dispatch(registerSuccess({user,pwd,type}))
      }else {
        dispatch(errorMsg(res.data.msg))
      }
      })
  }
}
```
5. 下一步就是向后台提交数据(post)
由第四步可知，通过该reducer创建的store中的初始状态值为initState，为了实现将注册页面保存的状态传到后端，这里将名为register的异步action creator通过connect组件传递给注册组件，因此，在点击注册按钮的时候，就将注册页面的信息传递给register函数，实现向后台提交数据，更新register.js如下：
```js
import {connect} from 'react-redux';//新加的
import {register} from '../../reduxs/user.redux';//新加的
import React from 'react';
import Logo from '../../component/logo/logo';
import {List, InputItem,Radio, WhiteSpace,Button} from 'antd-mobile';

@connect(
    state=>state.user,
    {register}
)//新加的
class Register extends React.Component{
    constructor(props){
        super(props)
        this.state={
          user: '',
          pwd: '',
          repeatpwd: '',
          type: 'genius'
        }
        this.handleRegister=this.handleRegister.bind(this)
    }
    handleChange(key,val){
      this.setState({
        [key]:val   //注意这里一定要使用方括号
        })
    }
    handleRegister(){
        this.props.register(this.state)//在这里实现了数据的提交
    }
    render(){
        const RadioItem=Radio.RadioItem;
        return (
            <div>
            <Logo></Logo>
                <List>
                    {this.props.msg? <p className="error-msg">{this.props.msg}</p>: null}//新加的一个简单的报错信息

                    <InputItem onChange={v=>this.handleChange('user',v)}> User Name </InputItem>
                    <WhiteSpace></WhiteSpace>
                    <InputItem type='password' onChange={v=>this.handleChange('pwd',v)}> Set password </InputItem>
                    <WhiteSpace ></WhiteSpace>
                    <InputItem type='password' onChange={v=>this.handleChange('repeatpwd',v)}> Confirm password </InputItem>
                    <WhiteSpace></WhiteSpace>
                    <RadioItem 
                    checked={this.state.type==='genius'}
                    onChange={()=>this.handleChange('type','genius')}>Niu Ren</RadioItem>
                    <WhiteSpace></WhiteSpace>
                    <RadioItem
                    checked={this.state.type==='BOSS'}
                    onChange={()=>this.handleChange('type','boss')}>BOSS</RadioItem>
                    <Button type='primary' onClick={this.handleRegister}>REGUSTER</Button>
                </List>
            </div>)
    }
}
export default Register
```
6. 向后端提交了数据之后，要实现跳转，也是由redux来进行跳转，因此在redux的初始状态中加入字段“redirectTo”;因为登录，注册后都要实现一个跳转;这个redirectTo是不能写死的，需要根据用户的身份以及信息是否完善的情况来进行跳转的，因此需要进行一些计算;因为大部分组件都涉及到路由的跳转，因此专门抽离出一个工具文件utils.js专门控制路由的跳转;
```js
//工具函数，专门用于页面跳转
export function getRedirectPath({type,avatar}){
    //根据用户信息。返回跳转页面地址
    // 根据user.type 类型跳转到boss页面和genius页面
    // 根据用户头像avatar 跳转到bossinfo /geniusinfo
    // 
    let url=(type==='boss')?'/boss':'/genius';
    if(!avatar){
        //表示没有头像信息，需要去完善用户信息（本项目中默认有了头像的情况，就代表已经完善了用户信息，就不需要跳转到完善信息页面去完善信息）
        url +='info'
    }
    return url
}
export function getChatId(userId,targetId){
    return [userId,targetId].sort().join('_')
}
```
因此在注册组件的logo  前面加上一个判断路由的语句：
{this.props.redirectTo ? <Redirect to={this.props.redirectTo}></Redirect> : null};
相应的在登录组件前面也添加一句：{this.props.redirectTo&&this.props.redirectTo!=='/login' ? <Redirect to={this.props.redirectTo}></Redirect> : null}
因为要向后端提交数据，需要准备redux，express，mongodb;故接下来主要实现后端的express和mongodb;
### 判断路由组件（AuthRoute）
因为需要根据身份，登录的状态，以及当前所处的位置进行检测，并作相应的跳转，比较麻烦，所以单独抽离出一个组件AuthRoute;<br />
该组件的目的主要是用于获取用户信息，并根据信息做相应的跳转;<br />
跳转是根据获取到的用户信息实现的;需要考虑登录状态，现在所在的url（login是可以不用跳转的），用户的身份（Boss or 牛人），用户是否完善信息等
我们利用在componentDidMount（）里面使用axios.get()方法获取用户信息，主要代码如下：
```js
componentDidMount(){
  axios.get('user/info')
  .then(res=>{
    if(res.status === 200){
      if(res.data.code==0){
        //有登录信息

      }else{
        this.props.history.push('/login') //未登录跳转到登录页面 ！！！注意这里AuthRoute不是路由组件，自身没有histor这个参数，需要从React-Router中引入withRouter
      }
      console.log(res.data)
    }
    })
}
```
#### 后端用户信息模拟）
因为AuthRoute组件需要获取用户的信息，因此需要先在后台模拟出用户的数据;此外，又因为与后端的数据交互交多，因此专门抽离出一个模块（user.js）,该模块放置的是与用户相关的express接口;如下是一个user.js的一部分

```js
const express = require('express')
const Router = express.Router()
Router.get('/info',function(req,res){
  return res.json({code:1})//code为1表示获取信息未成功
  })
module.exports = Router
```
又因为Express可以使用use（）处理中间件，因此可在后端入口文件server.js中利用use方法，设置一个url和对应的子路由名，用于获得用户信息
```js
const express = require('express')
const userRouter = require('./user')
app.use('user',userRouter)
```
* 通过mongodb实现数据模型model.js文件
```js
'use strict'
const mongoose = require('mongoose'); //引入mongoose模块

// 链接并使用imooc-chat这个集合 如果没有有imooc-chat这个集合，会自动新建一个
const DB_URL = 'mongodb://localhost:27017/imooc-chat';
mongoose.connect(DB_URL);//连接
const models={
    user:{
        'user': {'type':String,'require': true},
        'pwd': {'type': String,'require': true},
        'type': {'type':String,'require':true},
        //头像
        'avatar': {'type':String},
        // 个人简介或者职位简介
        'desc': {'type':String},
        //职位名
        'title': {'type':String},
        // 如果你是boss，还要加两个字段
        'company': {'type':String},
        'money': {'type':String}
    },
    chat: {
    }
}//设置相应的字段
//批量动态的生成相应的模型，相应的文档名为models里面定义的key值
for (let m in models){
    mongoose.model(m,new mongoose.Schema(models[m]))
}
module.exports={
    getModel: function(name){
        return mongoose.model(name)
    }
}
```
 * 引入了一个插件utility为cookie信息中的密码进行加密，也相应的进行了加盐操作
 ```js
 //该函数实现的功能是对加密进行加盐，使其变得更加难
function md5Pwd(pwd){
    const salt='xuejiao_is_great_678wxgywughd!@#RFDGFXJY~~' //这个可以自己定义
    return utils.md5(utils.md5(pwd+salt))
}
```
并且还在登录以及注册时进行post的时候，增加了查询条件（_filter），不希望密码被暴露出来！！！！
最后，设置cookie，写cookie是在res中写，读cookie是在request中;
## 完善信息（注册完之后）
两个完善信息的页面有：boss完善信息页面和牛人完善信息页面
首先，需要在入口文件设置相应的路由信息：
```js
<Route path='/bossinfo' component={BossInfo}></Route>
<Route path='/geniusinfo' component={GeniusInfo}></Route>
```
### BOSS页面
1. 引用NavBar从antd-mobile里面,作为完善信息页的Header
2. 选择头像（因为BOSS页面和牛人页面都有选择头像，因此抽离出选择头像功能作为一个组件）
3. 信息主要包括：招聘职位，公司名称，职位薪资，职位要求;主要从antd-mobile引入InputItem，和TextareaItem组件

  #### 头像组件
  1. 先从文件夹下，批量录入图片文件，并保存在一个变量中;
  2. 并从antd-mobile引入Grid组件用来显示图片;Grid是以宫格的形式显示图片。
  3. 给Grid添加onClick事件进行选择头像的交互;因此在Click事件中首先利用setState（）函数进行状态的改变，与此同时需要将选择的头像参数传给父组件传过来的函数，并且在这里使用了PropTypes插件来控制传入的参数的类型，即使用了属性类型校验
  4. 因为选择了头像需要显示出已经选择了哪一个头像，如果未选择则显示提醒‘请选择头像’，故引入List从antd-mobile里面，并将Grid组件也包括在里面;
4. 设置一个Button按钮，保存相应的用户信息，同时还是需要user.redux.js来管理，从user.redux.js里面传入一个update函数，用来传递用户完善的信息。
更新user.redux.js,在这里重新将登录成功，注册成功，以及完善信息成功这三个状态统一为一个action：AUTH_SUCCESS，并创建一个名叫update的action来进行post传送数据，并在这里过滤了pwd字段，使其不显示
### 牛人完善信息页面
大致和BOSS完善信息一样，只是输入的字段有些不一样，具体过程和一样
## 页面Dashboard
首先，在入口文件index.js设置好相应的路由信息：
```js
<Route component={Dashboard}></Route>
```
因为牛人列表，BOSS列表，消息列表，个人中心这四个页面都有相同的骨架，因此将这些组件需要一个外部Router对这四个组件进行包裹，然后根据对应的具体路由进行跳转;因此，dashboard需要根据相应的path和当前的url（this.props.location）相比较来渲染出响应的列表信息。因此，对于牛人列表页面和Boss列表中有一个hide（隐藏）字段
```js
 render(){
        const {pathname} = this.props.location//Dashboad是路由组件，不需要使用withRouter
        const user = this.props.user //获取用户信息
        // 定义了四个页面的相关路由信息，以及相应的信息
        const navList=[
            {
                path:'/boss',
                text:'牛人',//以boss身份进来的人，看到的是牛人列表
                icon:'boss',//要渲染页面的图标
                title:'牛人列表',//Header上面的title
                component:Boss,//要渲染的组件的名称
                hide:user.type==='genius'//不同的身份看到不同的页面，对于不让其看到的身份进行隐藏
            },
             {
                path:'/genius',
                text:'boss',//要查看的boss
                icon:'job',
                title:'BOSS列表',
                component:Genius,
                hide:user.type==='boss'
            },
             {
                path:'/msg',
                text:'消息',
                icon:'msg',
                title:'消息列表',
                component:Msg,
            },
            {
                path:'/me',
                text:'我',
                icon:'user',
                title:'个人中心',
                component:User,
            }
        ]
        return (
            <div>
                <NavBar 
                className='fixed-header'
                mode='dark'
                >{navList.find(v=>v.path===pathname).title}
                </NavBar>
                <div>
                <Switch>
                {
                    navList.map(v=>(
                        <Route key={v.path} path={v.path} component={v.component}></Route>
                    ))
                }
                </Switch>
                </div>
                <NavLinkBar data={navList}></NavLinkBar>

            </div>)
    }
}
```
其中NavBar是从antd-mobile组件中引用的，在这里相当于一个头部组件，而NavLinkBar组件是一个底部组件，该组件类似与我们qq聊天页面的那个底部框，当点击先面相应的图标时，会显示相应的页面信息，会判断当前的url并与传过去的navlist中的path信息做比较，根据结果进行相应的跳转。最后是中间的内容部分，是利用switch来根据相应的path来渲染相应的组件。这些组件有牛人列表组件，Boss列表组件，消息列表，个人中心。
### Boss身份所看到的是牛人列表
在牛人列表，我们作为Boss身份想要看到所有的牛人的信息，因此需要从后端获取相应的是牛人身份的用户信息，并把其相应的信息展示出来
1. 获取相关的用户信息，在这里是在生命周期函数componentDidMount函数中获取相应的信息
axios.get('/user/list?type=genius')
        .then(res=>{
            if(res.data.code===0){
               dispatch(userList(res.data.data))
            }
        }
2. 将获取到的用户信息展示到列表中，利用anti-mobile中的Card组件将用户信息展示出来
### 牛人身份所看到的是Boss列表
与开发牛人列表组件一样，流程也一样，只是看到的用户信息不一样。因此，可以将牛人列表和Boss列表中展示用户信息的部分抽离出来形成一个单独的组件（命名为usercard），与此同时将获取用户信息发送的请求利用redux来管理，因此，新建一个reducer（命名为：chatuser）


## 高阶组件（HOC）
即函数可以作为参数被传递或者是函数可以作为返回值输出
```js
function hello(){
  console.log("hello, Nice to meet you")
}

function WrapperHello (fn){
  retur n function(){
    console.log("before say hello")
    fn()
    console.log("after say hello")
  }
}
hello = WrapperHello(hello)
hello()
```
```js
其实，组件就是一个函数，比如：
class Hello extends React.Component {
    render(){
    return <h2>Hello,I love yogurts</h2>h2>
  }
}

创建一个装饰器
function WrapperHello(Comp){
  class WrapComp extends React.Component {
  render(){
  return (
      <div>
        <p>这是一个HOC高阶组件特有的元素</p>
        <Comp {...this.props}></Comp>
      </div>
  )
}
}
return WrapComp
}
Hello=WrapperHello(Hello)
```
在本项目中，写了一个小小的装饰器angle.form.js,该装饰其用来包裹登录组件和注册组件，使其自带handleChange事件。
## Socket.io
基于Web Sockets协议，是基于事件的实时双向通信库。事件进行双向通信，配合express，快速开发实时应用（端口API：on,once,emit）
* socket.io与ajax的区别
   
  * 准确地说，socket.io只是实现webSocket的一个库，ajax实现异步刷新，异步获取数据;
  * 基于不同的协议;ajax基于http协议，单向，实时获取数据只能通过轮询;socket.io基于WebSocket双向通信协议，后端可以推送数据;
* webSocket 协议与HTTP 协议的主要区别是可以双向发送数据，服务器端有新数据的时候，会自动推送数据。
* 在本项目中，socket.io 后端API（与express配合使用）
  ```js
  const app=express();
  const server=require('http').Server(app);
  const io=require('socket.io')(server);
  ...

  server.listen(9093,function(){
    console.log('Node app start at port 9093')
  })
  ```
* socket.io前端API（socket.io-client）
  因为前后端的端口不一样，则需要手动进行连接一下：
  ```js
   const socket=io('ws://localhost:9093')
  ```

### chat组件
1. 首先,在入口文件index.js中设置相应的路由
```js
<Route path='/chat/:user' component={Chat}></Route>
```
为什么是/chat/：user，因为我们可以从redux中获得当前用户的信息，但是需要知道聊天对象user
2. 实现聊天页面跳转，在牛人列表，或者是Boss列表，点击相应的用户卡片，即可进入聊天页面，实现跳转
3. 实现前后端联调
首先，在前端建立一个socket连接，并发送一个事件，然后在后端进行监听
```js
import io from 'socket.io-client';
const socket=io('ws://localhost:9093');//手动连接，因为有跨域
```
在后端server.js中：
```js
const app = express();//app是一个express实例
//work with express
const server=require('http').Server(app);

const io=require('socket.io')(server);
//io是全局的链接，传入的参数socket是当前的连接，io.on监听事件
// data是传过来的数据，socket是当前的请求，io是全局的请求
// 使用io将接收到的数据发送到全局

io.on('connection',function(socket){
    console.log('user login')   //表示已经和前端连接好了
    //在后端进行sendmsg的监听
    socket.on('sendmsg',function(data){
        // console.log(data);
        //向全局广播事件，表示已经接收到消息了
        io.emit('recvmsg',data)
        const from=data.from
        const to=data.to
        const msg=data.msg
        const chatid=[from,to].sort().join('_')
        Chat.create({chatid,from,to,content:msg},function(err,doc){
            io.emit('recvmsg',Object.assign({},doc._doc))
        })
        // console.log(data)
    })
})
```
4. 前面的聊天状态并没有入库，因此在后端建立聊天信息模型
```js
const models={
    user:{
    ...
    },
    chat: {
        'chatid':{'type':String,'require':true},//是两个人的id排序拼接起来的字符串
        'from':{'type':String,'require':true},//聊天的消息是来自谁
        'to':{'type':String,'require':true},//发给谁的
        'read':{'type':Boolean,'default':false},//未读的状态只对“to”有效
        'content':{'type':String,'require':true,'default':''},//聊天内容
        'create_time':{'type':Number,'default':new Date().getTime()} //发送信息的事件，以便聊天消息页面的展示
    }
}
```
  *注意：
  1. chatid是用于两人聊天的唯一标识;因为两个人的聊天信息是放在同一个列表中的，若无chatid，则需要查询两次，第一次是from是我;to表示你you;第二次是from是你，to是我。
  2. 在入口文件中设置路由的时候，url中带有聊天对象的id，而redux的状态中保存有当前登录用户的id。
5. 将聊天的请求事件放在redux里面统一进行管理，故新建一个reducer（chat.redux.js）
设置相应的action（这里设置了三个action：MSG_LIST(获取聊天列表),MSG_RECV(读取信息),MSG_READ(标识已读)），并且这些都是实时获取的;
6. 这一步：用户发送信息，传给socket，然后socket入库并广播全局
7.将收到的信息都展示在左边，自己发送的信息样式固定在右边，就像平时qq聊天一样
```js
const chatid=getChatId(userid,this.props.user._id)
const chatmsgs=this.props.chat.chatmsg.filter(v=>v.chatid==chatid)
{chatmsgs.map(v=>{
                    const avatar=require(`../img/${users[v.from].avatar}.png`)
                    return v.from==userid?(
                        <List key={v._id}>
                            <Item
                                thumb={avatar}
                            >
                                {v.content}
                            </Item>
                        </List>
                        ):(
                        <List key={v._id}>
                            <Item
                                extra={<img src={avatar} />}
                                className='chat-me'
                                >{v.content}
                            </Item>
                        </List>
                        )
                })}
```
8.未读消息的显示（在navlink.js中利用badge）
```js
根据路径过滤一下，否则下面的每个图标上都会显示未读数
badge={v.path=='/msg'? this.props.unread:0}
```
9.将用户对应的用户名和头像通过相应的查询条件从后端获取到
```js
//server.js
Router.get('/getmsglist',function(req,res){
    //只需要当前用户的信息。故利用cookis将所有的用户信息获取出来
    //{'$or':[{from:user,to:user}]}
    const user=req.cookies.userid
    User.find({},function(e,userdoc){
        let users={}
        userdoc.forEach(v=>{
            users[v._id]={name:v.user,avatar:v.avatar}
        })
        Chat.find({'$or':[{from:user},{to:user}]},function(err,doc){
        if(!err){
            return res.json({code:0,msgs:doc,users:users})
        }
    })
    })
})
```
```js
//前端获取数据（chat.redux.js）
export function getMsgList(){
    return (dispatch,getState)=>{
        axios.get('/user/getmsglist').then(
            res=>{
                if(res.status==200&&res.data.code==0){
                    const userid=getState().user._id
                    // console.log('getState',getState());
                    dispatch(msgList(res.data.msgs,res.data.users,userid))
                }
            })
    }
}
```
### Message消息列表的实现
首先，需要清楚，消息列表应该是以用户为单位进行统计;
然后，根据chatid，按照聊天用户分组
```js
const msgGroup={}
    this.props.chat.chatmsg.forEach(v=>{
        msgGroup[v.chatid]=msgGroup[v.chatid] || []
        msgGroup[v.chatid].push(v)
    })
```
其次，利用列表List对其进行渲染，并且列表上显示两个人聊天消息的最后一条消息;通过create_time字段来判断是否是最后一条消息;
这里要注意消息里面的列表信息有可能是当前用户发送给其他用户的，也有可能是其他用户发给当前用户的，因此from和to是不确定的，因此需要根据from与当前用户进行比较
```js
const targetId=v[0].from===userid?v[0].to:v[0].from;
```
然后，是未读消息的显示，也是需要对信息进行过滤，只统计其他用户发给当前用户的数量，且是未读的
```js
const unreadNum=v.filter(v=>!v.read&&v.to===userid).length
```
最后，应该根据时间将最信的消息分组展示在最顶部，因此需要根据发送的最后一条消息的时间来排序，将最新的消息分组渲染在最上面。
```js
const chatList = Object.values(msgGroup).sort((a,b)=>{
            const a_last=this.getLast(a).create_time
            const b_last=this.getLast(b).create_time
            return b_last-a_last
        })
```
### 消息未读数量更新
首先，需要知道在React-router4中，this.props中存储了一个match字段，表示当前匹配的路由信息;match里面有isExact字段，还有paramas字段，paramas里面包括path和url，其中path是我们在入口文件中定义的路由，而url是是实际的url。
我们希望当前的用户在点击了未读消息以后，则需要将read字段的值变为true，这就需要发起请求，并在后端修改相应的read字段，同时将修改的消息数量传递过来;
收到响应请求以后，利用dispatch事件派发相应的action，在reducer中更新消息数量。更新数量的时候也需要过滤数据，并不是将所有的未读数量都标为已读，需要根据to字段来进行消息数的更新
```js
chatmsg:state.chatmsg.map(v=>({...v，read:from===v.from?true:v.read}))//谁（from）发给我的，要和我们当前点进去的v.from相同，才将read字段表示为true（已读），如果不等与，则不改变
```

### 收尾优化
1、eslint代码校验工具
2、react16特有的错误处理机制
3、react性能优化
#### 性能优化
性能优化：react本身的性能优化（组件内部的性能优化和组件之间的优化），redux的优化（有些生成state的过程的计算），服务端渲染以及SSR，让首屏速度更快，以及懒加载等方式
* 组件优化：1、this的绑定;2、尽量避免在render中定义变量，属性传递的时候尽量少传且注意不要直接定义变量
* 多组件优化：定制shouldComponentUpdate，或者使用相应的库;插件PureComponent针对组件内部无状态，只是从父组件传递参数时非常有用，它只进行简单的浅层比较。
而reselect库相当与是将从state中获取数据，和将获取到的数据变成组件内部可用的数据分成了两步，内部会做一些缓存，因此相对于性能会比较好一点
* 遍历数组是一定要加key
#### eslint代码校验工具
可配置一些个人的配置;

```js
"eslintConfig": {
    "extends": "react-app",
    "rules":{
      "eqeqeq":["off"],
      "jsx-a11y/img-has-alt":[0] //0也表示是关闭的意思
    }
  },
```
##### 服务器端渲染
1. 需要node环境支持es6语法，需要使用babel-node;故安装插件：npm install babel-cli --save;
2. 修改配置，新建一个文件（.babelrc）对babel进行配置,使后端和前端一样支持React组件的书写形式;
```js
//.babelrc
 {
    "presets": [
      "react-app"
    ],
    "plugins": [
      [
        "import",
        {
          "libraryName": "antd-mobile",
          "style": "css"
        }
      ],
      "transform-decorators-legacy"
    ]
  }
  ```
  3. 修改server.js文件，使用import代替requre引入文件
  4. 使用react-dom-Server来做服务端渲染，引入{renderToString} from react-dom/server;目的是使react组件可以被渲染Dom树;
  5. 为了使后端支持css以及图片文件，需要引入两个库：
  npm install css-modules-require-hook --save
  npm install asset-require-hook --save
  6. 修改server.js,完成服务器端渲染:
  ```js
  import express from 'express';//引入express模块
  import utils from 'utility';
  import bodyParser from 'body-parser';//用于解析post过来的json
  import cookieParser from 'cookie-parser';//用于解析cookie
  import model from './model';
  import path from 'path';
  //https://github.com/css-modules/css-modules-require-hook
  import csshook from 'css-modules-require-hook/preset' // import hook before routes
  import assethook from 'asset-require-hook';
  assethook({
      extensions:['png']
  })

  import React from 'react';
  import { createStore, applyMiddleware, compose} from 'redux';
  import thunk from 'redux-thunk';
  import {Provider} from 'react-redux';
  import {StaticRouter} from 'react-router-dom';
  import App from '../src/App';
  import reducers from '../src/reducers';


  import {renderToString} from 'react-dom/server';
  import staticPath from '../build/asset-manifest.json';//获取静态资源文件，因为文件在变化，需要动态的引入
  // console.log(staticPath)

  const Chat = model.getModel('chat');
  const app = express();//app是一个express实例
  const server=require('http').Server(app);//work with express
  const io=require('socket.io')(server);

  //io是全局的链接，传入的参数socket是当前的连接，io.on监听事件
  // data是传过来的数据，socket是当前的请求，io是全局的请求
  // 使用io将接收到的数据发送到全局

  io.on('connection',function(socket){
      console.log('user login')//说明用户已经
      socket.on('sendmsg',function(data){
          // console.log(data);
          io.emit('recvmsg',data)//发送全局事件
          const from=data.from
          const to=data.to
          const msg=data.msg
          const chatid=[from,to].sort().join('_')
          Chat.create({chatid,from,to,content:msg},function(err,doc){
              io.emit('recvmsg',Object.assign({},doc._doc))
          })
          // console.log(data)
      })
  })

  const userRouter = require('./user');

  app.use(cookieParser())    //  先解析cookie
  app.use(bodyParser.json())   //解析post传过来的json
  //开启一个中间件,若中间件是路由的话，在前面就用路由的形式,userRouter是一个子路由的形式
  app.use('/user',userRouter);
  // app.listen(9093,function(){
  //     console.log('Node app start at port 9093');
  // })

  //设置静态资源地址
  app.use(function(req,res,next){
      if(req.url.startsWith('/user/')||req.url.startsWith('/static/')){
          return next()
      }
      const store=createStore(reducers,compose(
      applyMiddleware(thunk),
      ))
      let context = {}
      const markup=renderToString(
          (<Provider store={store}>
              <StaticRouter
                  location={req.url}
                  context={context}>
                  <App></App>
              </StaticRouter>
          </Provider>)
          )
      const obj={
          '/msg':'React聊天消息页面',
          '/boss':'Boss查看牛人列表页'
      }
      const pageHtml = `
          <!doctype html>
          <html lang="en">
            <head>
              <meta charset="utf-8">
              <meta name="viewport" content="width=device-width, initial-scale=1">
              <meta name="theme-color" content="#000000">
              <meta name="keywords" content="React,Redux,Chat,SSR">
              <meta name="description" content='${obj[req.url]}'>
              <title>React App</title>
              <link rel="stylesheet" href="/${staticPath['main.css']}" />
            </head>
            <body>
              <div id="root">${markup}</div>
              <script src="/${staticPath['main.js']}"></script>
            </body>
          </html>
      `
      // return res.sendFile(path.resolve('build/index.html'))
      res.send(pageHtml)
  })
  app.use('/',express.static(path.resolve('build')))
  server.listen(9093,function(){
      console.log('Node app start at port 9093');
  })
 ```
 ## React16 新特性
 * 新版本带来的新特性和新功能呢个
 1. 新的核心算法Fiber
 2. Render更为灵活，可以返回数组，字符串
 3. 错误处理机制
 4. Portals组件（React 可在DOM节点之外渲染元素，比如弹窗的效果）
 5. 更好更快的服务端渲染
 6. 体积更小，MIT协议（完全开源）
 7. 还增加了一个新的生命周期函数：componentDidCatch（）;可用于处理错误
 * 新版本更快的流失渲染
 1. 之前版本的renderToString 解析为字符串;
 2. 新版本的renderToNodeStream 解析为可读的字节流对象;
 3. 并使用ReactDom.hydrate取代render
* 使用renderToNodeStream来进行服务器端渲染
```js
//server.js
app.use(function(req,res,next){
    if(req.url.startsWith('/user/')||req.url.startsWith('/static/')){
        return next()
    }
    const store=createStore(reducers,compose(
    applyMiddleware(thunk),
    ))
    const obj={
        '/msg':'React聊天消息页面',
        '/boss':'Boss查看牛人列表页'
    }
    let context = {}
    res.write(`<!doctype html>
        <html lang="en">
          <head>
            <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1">
            <meta name="theme-color" content="#000000">
            <meta name="keywords" content="React,Redux,Chat,SSR">
            <meta name="description" content='${obj[req.url]}'>
            <title>React App</title>
            <link rel="stylesheet" href="/${staticPath['main.css']}" />
          </head>
          <body>
            <div id="root">`)
    const markupStream = renderToNodeStream(
        (<Provider store={store}>
            <StaticRouter
                location={req.url}
                context={context}>
                <App></App>
            </StaticRouter>
        </Provider>)
        )
    markupStream.pipe(res,{end:false})
    markupStream.on('end',()=>{
        res.write(`</div>
            <script src="/${staticPath['main.js']}"></script>
          </body>
        </html>`)
        res.end()
    })
})
```
# Summary
- redux逻辑（项目中使用了三个redux逻辑）
  - user.redux.js 负责登录，注册，路由组件的action主要是异步发送请求以及从后端获取信息利用 reducer进行相应状态的更新
  - chat.redux.js 负责聊天功能的action与reducer，主要是异步发送数据，并向后端提交信息以及从后端获取信息
  - chatUser.redux.js 负责boss列表页面，牛人列表页面的action和reducer
- 支持聊天整个功能实现的主要action，以及对应的后端处理
  - 前端主要逻辑
    - MsgList：利用axios向后端发送get请求;传入的参数是相关的路由;收到相关的响应消息后做相应的处理（返回后并派发相应的action，并将返回的res信息传入到相应的action函数中，并且传入到action函数中的参数还包括getState()中得到的userid）;
    - MsgRecv：通过socket链接的on方法监听recvmsg事件，传给对应的action参数也是后端响应中的信息;
    - MsgSend：利用socket.emit('sendmsg',{from,to,msg})
    - MsgRead：利用axios向后端发送post请求，发送请求中传入的参数有：from的id，当前登录用户id，具体的msg;传给相应的action的参数为（userid，from，num：res.data.num）
  - 后端主要对应的逻辑
    - Router.get('/getmsglist',function(req,res){//首先，通过cookie获取到当前登录用户的userid;//利用User数据库模型的find函数+条件查询; 将用户的id作为键名，键值由name和头像组成的对象，并在chat字段中查找from和to字段都为user的消息集合并返回给前端})
    - 在io.on中监听connection，接受到前端的socket链接，并利用socket.on方法监听sendmsg事件，当监听到sendmsg事件后，利用chat字段的create方法把sendmsg中传递过来的from，to，msg参数保存在数据库中，与此同时，利用全局链接的emit方法发送一个recvmsg事件，并把对应发送过来的文档的信息传入，进行合并
    - Router.post('/update',function(req,res){//通过cookie获取到当前用户的id;并把提交过来的from参数从body提取出来;//利用chat字段的update方法进行更新})
- Q&A
  - 在消息页面聊天时同步消息的解决方案;若未解决会出现同步时看过的消息，需要再进入页面时，再进行更新
  - 解决方案有两种：1、只要在当前path中，进行更新;2、将readMsg（）从之前的componentDidMount（）声明周期函数中迁移到componentWillUnMount()生命周期函数中
