---
layout: post
title:  "[React] create-react-app으로 프로젝트 시작하기"
date:   2018-06-05 12:44:06 +0900
author: 방구석엔지니어
categories: react
tags:	react create-react-app
cover:  "/assets/react.jpeg"
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

```jsx
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

```jsx
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
render 함수는 화면에 그려지는 엘리먼트들을 jsx 문법으로 리턴한다. 우선 할 일을 등록하는 양식이 필요할 것 같다. 간단하게 할 일을 입력할 수 있는 ```input```과 ```button```을 만들어보자. 스타일도 포기할 수 없으니 css는 [bootstrap](https://getbootstrap.com)을 사용하도록 해보자.

### public/index.html ###
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    ...
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css" integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    ...
  </head>
  ...
</html>

```

```public/index.html```에 bootstrap css를 추가한다.

### src/App.js ###
```jsx
...

let todoId = 0 // todo 아이템의 id를 증가시키며 저장하는 변수

class App extends Component {
  constructor (props) {
    super(props)
    this.state = {
      todoItems: [],
      todoInput: '' // 새 할 일 input의 value를 저장하는 state
    }
    this._handleOnClickAddItem = this._handleOnClickAddItem.bind(this)
    this._handleOnChangeTodoInput = this._handleOnChangeTodoInput.bind(this)
  }

  _handleOnClickAddItem () {
    const { todoInput, todoItems } = this.state
    if (todoInput.length === 0) return alert('내용을 입력해주세요.')
    const todoItem = {
      id: todoId++,
      title: todoInput,
      isCompleted: false
    }
    const newTodoItems = todoItems.slice(0)
    newTodoItems.push(todoItem)
    this.setState({ todoItems: newTodoItems, todoInput: '' })
  }

  _handleOnChangeTodoInput (e) {
    this.setState({ todoInput: e.target.value })
  }

  render () {
    return (
      <div className='container' style={% raw %}{{ maxWidth: 600, padding: '20px 0' }}{% endraw %}>
        <div className='row'>
          <div className='col text-center'>
            <div className='input-group'>
              <input
                type='text'
                className='form-control'
                placeholder='새로운 할 일을 입력해주세요.'
                value={this.state.todoInput}
                onChange={this._handleOnChangeTodoInput}
                onKeyDown={e => e.keyCode === 13 ? this._handleOnClickAddItem() : null}
              />
              <div className='input-group-append'>
                <button
                  className='btn btn-outline-secondary'
                  onClick={this._handleOnClickAddItem}
                >
                  등록
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    )
  }
}

export default App;
```
새 할일을 등록할 폼을 작성해보았다. 이 코드만으로도 리액트의 여러가지 문법들과 보편적인 구조를 확인해볼 수 있다.

- 스타일링 : jsx 문법에서의 클래스 명은 ```className```이라는 속성을 사용한다. es6의 ```class```지정자와 겹치기 때문이다. 따라서 기존 레가시 html을 복사 붙여넣기 할 경우 class 속성이 그대로 남아있으면 스타일이 먹지 않으므로 주의해야 한다. 인라인 스타일의 경우에는 javascript object 형태로 입력해야 한다. jsx 문법에서 javascript를 삽입하기 위해서는 중괄호 ```{}```를 사용하는데, ```style```속성 내에 javascript object를 넣다 보니 ```style={% raw %}{{ maxWidth: 600, padding: '20px 0' }}{% endraw %}```과 같은 형태가 된다. css 문법의 케밥케이스를 카멜케이스로 옮기는 방식으로 키를 지정하고, 값은 css 문법 그대로 string 타입으로 넣거나, number타입으로 넣어도 된다. number타입일 경우에 단위는 픽셀로 인식이 된다.

- 폼 엘리먼트 : 리액트 컴포넌트는 기본적으로 state와 props의 변화에 따라 새롭게 화면을 랜더링을 한다. (메모리에서 DOM을 비교하여 변경이 되는DOM만 새로 렌더링) ```input```은 사용자의 키보드 입력이 들어올 때 마다 화면을 새로 렌더링해줘야 하는데, 이를 위해 컴포넌트의 state를 이용한다. 위 예제에서는 ```todoInput```이라는 state를 만들어주었고, 초기값을 ```''```로 설정했다. 그리고 ```input```태그의 ```value```속성에 ```todoInput```을 할당해주었다. 또한 ```input```태그의 ```onChange```이벤트에 ```this._handleOnChangeTodoInput```메소드를 할당하여 키보드 입력 시에 ```todoInput``` state를 변경하도록 했다. 리액트에서 하나의 ```input```태그가 작동하려면 이러한 일련의 과정을 거쳐야한다. (조금 복잡하다고 생각할 수도 있지만 간편하게 사용할 수 있는 폼 컨트롤과 관련된 라이브러리들을 사용하면 되므로 너무 걱정하지 말자. 원리를 이해하기 위한 과정일 뿐이다.)

- 이벤트바인딩 : ```input```에서 ```onChange```속성을 이용하여 DOM과 이벤트를 바인딩한 것 처럼, jsx 태그에 ```on****```와 같은 속성을 넣어주는 것으로 이벤트와 함수를 바인딩시킬 수 있다. ```등록```버튼에 ```onClick```속성을 넣어주어 버튼 클릭 시에 새로운 할 일을 등록하는 함수를 실행한다. 그리고 ```input```태그에 ```onKeyDown```속성을 넣어주어 키보드 입력이 엔터키일경우 같은 기능을 수행하도록 했다. class의 멤버 함수를 선언할 때 함수 안에서 ```this```를 사용하는 경우 ```constructor```에 ```this._handleOnClickAddItem = this._handleOnClickAddItem.bind(this)```와 같이 함수에 인스턴스를 바인딩 해주는 것을 잊지말자. (해줘야 하는 경우와 안해도 되는 경우가 있는데, 자세한 설명은 [이곳](https://medium.freecodecamp.org/react-binding-patterns-5-approaches-for-handling-this-92c651b5af56)을 참고하자.)

```jsx
{% raw %}
import React, { Component } from 'react';

let todoId = 0 // todo 아이템의 id를 증가시키며 저장하는 변수

class App extends Component {
  constructor (props) {
    super(props)
    this.state = {
      todoItems: [],
      todoInput: ''
    }
    this._handleOnClickAddItem = this._handleOnClickAddItem.bind(this)
    this._handleOnChangeTodoInput = this._handleOnChangeTodoInput.bind(this)
    this._handleOnClickToggleState = this._handleOnClickToggleState.bind(this)
    this._handleOnClickRemove = this._handleOnClickRemove.bind(this)
  }

  _handleOnClickAddItem () {
    const { todoInput, todoItems } = this.state
    if (todoInput.length === 0) return alert('내용을 입력해주세요.')
    const todoItem = {
      id: todoId++,
      title: todoInput,
      isCompleted: false
    }
    const newTodoItems = todoItems.slice(0)
    newTodoItems.push(todoItem)
    this.setState({ todoItems: newTodoItems, todoInput: '' })
  }

  _handleOnChangeTodoInput (e) {
    this.setState({ todoInput: e.target.value })
  }

  _handleOnClickToggleState (index) { // index에 해당하는 아이템의 isCompleted 를 토글한다.
    const { todoItems } = this.state
    const newTodoItems = todoItems.slice(0)
    newTodoItems[index].isCompleted = !todoItems[index].isCompleted
    this.setState({ todoItems: newTodoItems })
  }

  _handleOnClickRemove (id) {
    const { todoItems } = this.state
    const newTodoItems = todoItems.filter(item => item.id !== id)
    this.setState({ todoItems: newTodoItems })
  }

  render () {
    const renderCancelButton = item => (
      <button
        className='btn btn-danger btn-sm'
        style={{ marginLeft: 5 }}
        onClick={() => this._handleOnClickRemove(item.id)}
      >
        삭제
      </button>
    )
    return (
      <div className='container' style={{ maxWidth: 600, padding: '20px 0' }}>
        <div className='row'>
          <div className='col text-center'>
            <div className='input-group'>
              <input
                type='text'
                className='form-control'
                placeholder='새로운 할 일을 입력해주세요.'
                value={this.state.todoInput}
                onChange={this._handleOnChangeTodoInput}
                onKeyDown={e => e.keyCode === 13 ? this._handleOnClickAddItem() : null}
              />
              <div className='input-group-append'>
                <button
                  className='btn btn-outline-secondary'
                  onClick={this._handleOnClickAddItem}
                >
                  등록
                </button>
              </div>
            </div>
          </div>
        </div>
        <div className='row' style={ { marginTop: 20 } }>
          <div className='col-6'>
            <h3>해야할 일</h3>
            {
              this.state.todoItems.filter(item => !item.isCompleted).map(item =>
                <div key={item.id} style={{ margin: 10 }}>
                  <span style={{ marginRight: 5 }}>- {item.title}</span>
                  <button
                    className='btn btn-success btn-sm'
                    onClick={() => this._handleOnClickToggleState(item.id)}
                  >
                    완료
                  </button>
                  {renderCancelButton(item)}
                </div>
              )
            }
          </div>
          <div className='col-6'>
            <h3>완료한 일</h3>
            {
              this.state.todoItems.filter(item => item.isCompleted).map(item =>
                <div key={item.id} style={{ margin: 10 }}>
                  <span style={{ marginRight: 5 }}>- {item.title}</span>
                  <button
                    className='btn btn-warning btn-sm'
                    onClick={() => this._handleOnClickToggleState(item.id)}
                  >
                    취소
                  </button>
                  {renderCancelButton(item)}
                </div>
              )
            }
          </div>
        </div>
      </div>
    );
  }
}

export default App;
{% endraw %}
```

등록된 최종적인 소스이다. ```render```메소드 안에서 중복되는 엘리먼트들은 ```renderCancelButton```함수처럼 함수로 지정하여 사용할 수도 있다. 하지만 이 방법은 ```render```함수가 호출될 때 마다(화면을 다시 렌더링 할 때 마다) 쓸 데 없이 함수를 다시 선언하는 과정을 거치니 앞으로는 멤버함수(```this._renderCancelButton```와 같은 방식)로 선언해주도록 하자. 아래와 같이 잘 작동되면 성공!!

![실행화면](/assets/posts/create-react-app_TODO_1.gif)

다음 포스팅에서는 내부 엘리먼트들을 리액트 컴포넌트로 분리해보도록 하겠다.
전체 소스는 [이곳](https://github.com/eunvanz/create-react-app-todo-1)에 공유되어있다.