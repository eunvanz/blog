---
layout: post
title: "[React] create-react-app으로 프로젝트 시작하기"
date: 2018-06-08 12:44:06 +0900
author: 방구석엔지니어
categories: react
tags: react create-react-app
cover: "/assets/react.jpg"
---

# create-react-app 이란? #
create-react-app은 페이스북에서 만든 react 웹 개발용 boilerplate이다. create-react-app이 나오기 전 까지는
1. 직접 모든 환경을 설정하거나, 
2. 남이 만든 boilerplate를 사용

했어야 했다. react는 es6 버전의 javascript로 작성하는 것이 일반화 되어있기 때문에 [webpack](https://webpack.js.org)이라는 모듈번들러로 컴파일 및 빌드 하는 것이 필수라 이 환경을 세팅해줘야한다. 즉, webpack도 공부해야 한다는 의미인데, 이게 쉽지가 않다. (react 공부하기에도 바쁘다.) 그래서 2번 방법을 주로 사용하곤 했는데, 개인이 만든 bolierplate이기 때문에 지속적 업데이트가 되지 않아 react 버전이 상승하면 익숙했던 bolierplate는 쓰지 못하게 되고, 새로운 bolierplate를 찾아야만 했다.

create-react-app은 페이스북에서 만들고 지속적으로 업데이트 되는 공식적인 boilerplate이기 때문에 위와 같은 걱정이 없어졌다. 이 포스팅에서는 create-react-app을 통해서 간단한 TODO 웹을 만들어 보도록 하겠다.

# create-react-app 설치 #
우선 npm 최신버전이 설치되어있어야 한다. 프로젝트 폴더를 생성하고자 하는 폴더로 가서 아래와 같이 명령어를 입력하자. npx는 npm 패키지를 로컬에 글로벌로 설치하지 않고 바로 일회성으로 실행할 수 있게 해주는 도구이다. npm 5.2.0 버전 이후부터 기본으로 제공된다.

```
npx create-react-app react-todo
```

npx가 실행이 안되는 구버전이라면 아래와 같이 해야한다.

```
npm install -g create-react-app
create-react-app react-todo
```

설치가 완료되면 하위에 ```react-todo``` 라는 폴더가 생성되어 있다. 폴더 이동 후 프로젝트를 로컬에서 실행해볼 수 있다.

```
cd react-todo
npm start
```

실행을 하면 브라우저가 실행되면서 ```localhost:3000```포트에 프로젝트가 떠 있는 것을 확인해볼 수 있다. (3000번 포트를 이미 사용 중이라면 3001, 3002, ...와 같이 증가된 포트에 뜬다.)

# create-react-app 프로젝트의 구조 #
폴더는 아래와 같이 구성되어있다.
```
react-todo/
  README.md
  node_modules/
  package.json
  public/
    index.html
    favicon.ico
  src/
    App.css
    App.js
    App.test.js
    index.css
    index.js
    logo.svg
```
src 폴더에 우리가 직접 작성할 소스코드 파일이 들어가고, public 폴더에 static 자원이 위치한다. ```public/index.html```과 ```src/index.js```는 엔트리 포인트가 되는 소스로, 파일이름이 변경되면 create-react-app은 작동하지 않으므로 주의한다.


```
{
  "name": "react-todo",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^16.4.0",
    "react-dom": "^16.4.0",
    "react-scripts": "1.1.4"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```
```package.json```파일을 얼여보면 이상하리 만큼 단순한 것을 확인할 수 있다. dependency가 3개 밖에 없고, devDependency도 없다. 하지만 ```node_modules/```폴더를 보면 알 수 있듯이, 내부적으로는 무수히 많은 dependency가 존재한다. 이는 ```create-react-app```의 특징인데, 모든 설정들이 캡슐화 되어있다. 사용자로 하여금 설정들을 건들지 못하도록 실제 ```package.json```을 숨겨두고, ```webpack```설정과 관련된 부분들도 모두 숨겨두었다. 한마디로 개발자들이 소스코드에만 집중할 수 있도록 배려해 놓았다. ```scripts```설정에 보면 사용할 수 있는 명령어 중에 ```eject```라는 명령어를 통해 이러한 봉인을 해제할 수 있으나, 이 포스팅에서는 ```create-react-app```의 의도대로 소스코드에만 집중해보도록 하겠다.

# 소스코드 작성하기 #
## 기존 소스코드 분석 ##
엔트리 포인트인 ```src/index.js```파일을 열어보자.
```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import registerServiceWorker from './registerServiceWorker';

ReactDOM.render(<App />, document.getElementById('root'));
registerServiceWorker();
```
코드는 ```javascript ES6``` 버전으로 작성되어 있고, ```jsx```문법도 섞여있다. 이 문법들에 대해 익숙하지 않다면 [이곳](https://jsdev.kr/t/es6/2944)과 [이곳](https://velopert.com/867)을 참고하자. ```ES6```문법은 익숙해지면 꽤나 편하게 사용할 수 있고, ```jsx```문법은 ```html```에 익숙하다면 그리 어렵지 않게 사용할 수 있다.

```
import React from 'react';
```
```jsx```문법을 사용하기 위해서는 ```react```모듈을 import 해야한다. 모든 react 컴포넌트에 필수적인 코드이다.

```
import ReactDOM from 'react-dom';
```
```react-dom```모듈은 react 앱을 최초 렌더링 하기위해 엔트리 포인트에서 쓰인다. 브라우저 뿐만 아니라 서버사이드용 랜더링 메소드를 지원한다.

```
import './index.css';
```
css파일을 import 구문으로 가져오고 있다. 이는 webpack의 [css-loader](https://github.com/webpack-contrib/css-loader)를 활용한 것인데, ```create-react-app```에서 기본적으로 세팅이 되어있다.

```
import App from './App';
```
```App```이라는 react 컴포넌트를 가져오는 코드이다. 컴포넌트는 react 웹에서 기본적인 화면을 구성하는 단위이다. 예를들어, 회원 가입 화면에서 button, input, textarea와 같은 것들이 컴포넌트이고, 이러한 컴포넌트들이 구성한 화면 조차도 하나의 컴포넌트이다. react는 이러한 컴포넌트들을 만들고, 조립하는 것을 용이하게 해줌으로써 개발에 편의성을 제공한다. 컴포넌트에 대해서는 뒤에서 더 자세히 설명하겠다.

```
import registerServiceWorker from './registerServiceWorker';
```
[service worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)란 네트웍이 느리거나 오프라인인 상태에서도 온라인 인것 처럼 리소스들을 캐싱해서 보여주는 모듈을 뜻한다. ```create-react-app```에서는 기본으로 구현하여 소스코드에 포함되어있다.

```
ReactDOM.render(<App />, document.getElementById('root'));
registerServiceWorker();
```
```root```라는 id를 가진 태그를 찾아서 그 안에 ```App```컴포넌트를 렌더링시킨다. 이곳이 우리가 만드는 react 웹의 엔트리 포인트인데, 껍데기가 되는 html 파일은 ```public/index.html```파일이 된다. 이 파일을 열어보면 ```root```라는 id를 가진 ```div```태그가 존재함을 확인할 수 있다.

그렇다면 우리가 소스코드를 작성해야 하는 파일은? ```App.js```파일이 된다. 파일을 열어보자.

```
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title">Welcome to React</h1>
        </header>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
      </div>
    );
  }
}

export default App;
```
모든 react 컴포넌트들은 react 모듈의 ```Component```클래스를 상속받는다. ```Component```는 ui를 구성하는 엘리먼트들을 독립적이고 재사용 가능하게 만드는 추상 클래스라고 생각하면 된다. 미리 정의해 둔 메소드들이 있고, 그 메소드들을 override 해서 컴포넌트들을 구현하는데, 그 중에서 화면을 그려주는 ```render``` 메소드는 필수적으로 정의해야 한다. ```render``` 메소드는 반드시 jsx 문법으로 작성한 하나의 엘리먼트를 리턴한다.

## state 만들기 ##
TODO 웹을 만들기에 앞서서, react Component에서 빠질 수 없는 개념인 ```state```에 대해 알고 넘어가고자 한다. ```state```는 컴포넌트의 스코프 안의 지역변수라고 할 수 있다. 이 state가 변경되면 이 state를 참조하고 있는 컴포넌트의 다른 부분들도 영향을 받아 업데이트 된다. 우선 state를 선언부터 해보겠다.

```
...

class App extends Component {
  constructor (props) {
    super(props)
    this.state = {
      todoItems: [] // todo 아이템이 들어갈 배열 선언
    }
  }
  render() {
    ...
  }
}

...
```
render 함수 위에 constructor를 선언하고, constructor 안에 ```this.state```를 선언했다. state는 기본적으로 Object의 형태로 선언한다. 여기에서는 todo 아이템이 들어갈 배열을 선언했다. 이 state에 ```todo``` 객체들을 넣을 생각이다.

## render 함수 구현하기 ##
render 함수는 화면에 그려지는 엘리먼트들을 jsx 문법으로 리턴한다. 우선 할 일을 등록하는 양식이 필요할 것 같다.

 

