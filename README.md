# @mochang2/react

jQuery 쓰던 시절과 달리 React(Vue, Svelte 등도)는 DOM API를 쓸 필요가 없게 만들었다.  
이를 '선언적'이라고 표현한다.  
React는 문법에 맞춰 state만 관리하면 DOM API를 내부적으로 처리하여 사용자가 원하는 view를 손쉽게 만들 수 있도록 도와준다.

### 바닐라로 React(함수형 컴포넌트) 구현하기

해당 프로젝트는 React의 핵심 기능 5가지

1. Virtual DOM
2. JSX
3. RealDOM 렌더링
4. Diffing update(현재는 text가 변경된 경우로만 한정)
5. `useState` hook(메모리 최적화는 없음)
6. `useEffect` hook(dependency는 null을 제외한 primitive type 1개만 넣을 수 있음. clean up 제외)

를 구현했다.

다만 모든 html element에 props를 전달할 수 있게 코드를 짜면 작성해야 할 코드 양이 너무 방대해져서 생략했다.  
state에 대한 변화나 이벤트를 달기 위해서는 아래처럼 timer API나 `addEventListener`를 사용해야 한다.

```js
const Component = () => {
  // 렌더링될 때까지 기다림
  setTimeout(() => {
    document.querySelector('button').addEventListener('click', () => {
      alert('경고!');
    });
  }, 1000);

  return <button>click!</button>;
};
```

### 프로젝트 세팅 및 시작

(프로젝트를 js로 진행한다고 가정)

1. `npm install @mochang2/react && npm install -D @babel/core @babel/preset-env @babel/preset-react babel-loader html-webpack-plugin webpack webpack-cli webpack-dev-server`

2. `package.json` 설정

```json
{
  // ...
  "scripts": {
    "dev": "webpack serve --mode development",
    "build": "webpack --mode production"
  }
  // ...
}
```

3. `webpack.config.js` 설정.

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: __dirname + '/dist',
    filename: 'bundle.js',
  },
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env', '@babel/preset-react'],
          },
        },
      },
    ],
  },
  resolve: {
    extensions: ['.js'],
  },
  devServer: {
    port: 5000,
    hot: true,
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
    }),
  ],
};
```

4. `index.html` 선언.

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <script type="module" src="./index.js"></script>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

5. 컴포넌트 파일 구현. 파일 최상단에 바벨이 사용할 createElement 명시.

```js
// App.js
/* @jsx createElement */

import { createElement, useState, useEffect } from '@mochang2/react';

const CoffeeList = () => {
  const [coffees, setCoffees] = useState([
    { title: '에스프레소' },
    { title: '아메리카노' },
  ]);

  useEffect(() => {
    setTimeout(() => {
      setCoffees([{ title: '아메리카노' }, { title: '에스프레스' }]);
    }, 2000);
  }, []);

  return (
    <ul>
      {coffees.map(({ title }) => (
        <li>{title}</li>
      ))}
    </ul>
  );
};

export default CoffeeList;
```

6. `index.html`에서 참조하고 있는 `index.js` 구현.

```js
/* @jsx createElement */

import { createReactRoot, createElement, renderRealDOM } from '@mochang2/react';
import CoffeeList from './app.js';

const root = createReactRoot(document.querySelector('#root'));
root.render(CoffeeList);
```