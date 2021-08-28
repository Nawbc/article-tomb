插件收录在react-mini-plugins(素质三联，支持下小萌新)

sewerganger/react-mini-plugins
​github.com/sewerganger/react-mini-plugins

问题由来
css在前端一直是个棘手的问题，分开写总感觉有些麻烦, 所以出现了像

https://www.npmjs.com/package/styled-components
​www.npmjs.com/package/styled-components
这种包，但从我个人角度来讲styled-component感觉还是麻烦了些，所以呢写了个mini plugin component-style　是不是很山寨(滑稽)

使用
npm install component-style
Api

comptStyle(componentName: string, style: object) 参数一是模块名 需要是css selector，最好用id，参数二是个可嵌套的css 对象
import 'component-style/polyfill' StyleSheets.insertRule 的垫片
例子(不一定非要用在react)
import ReactDOM from 'react-dom';
import { comptStyle } from 'component-style';
import 'component-style/polyfill' // add polyfill to support ie5-8
import React from 'react';

function Demo(props) {
 comptStyle('#demo', props.comptStyle);
 return (
  <div id="demo">
    <div className="a">
     <div className="aa">
       component
      <div className="ab" />
     </div>
    <div className="b" />
    <div className="c" />
  </div>
 );
}

const demoStyle = {
 width: '100px',
 color: '#000',
 '.a': {
    width: '500px',
    height: '500px',
    background: 'red',
    '.aa': {
     'font-size': '20px'
    },
   '.ab': {
     width: '50px',
     height: '50px',
     background: 'yellow'
    }
  },
  '.b': {
    width: '200px',
    height: '200px',
    background: 'blue'
   },
  '.c': {
    width: '40px',
    height: '40px',
    background: 'green'
   }
 }}
}

function App() {
 return <Demo comptStyle={demoStyle} />;
}

/*eslint-env browser*/
ReactDOM.render(<App />, document.getElementById('root'));
原理
主要使用了StyleSheets.insertRule api 和一点简单的bfs算法


最后
style的接口还待优化下，自动提示不好, 能用就行