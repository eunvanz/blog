---
layout: post
title: "[React] Higher Order Component로 form handling 관심사 분리하기"
date: 2018-02-12 23:44:40 +0900
author: 방구석엔지니어
categories: react
tags: react hoc
cover: "/assets/react.jpeg"
---

[지난 포스팅][지난포스팅]에서 Higher Order Component(이하 HOC)에 대해 간략하게나마 알아보았다. 오늘은 좀 더 심화되고 실전적인 예시를 보여주려 한다. 실제로 내가 프로젝트를 진행하면서 부딪혔던 상황에서 어떠한 방식으로 HOC를 적용하여 문제를 해결했는지 좀 더 단순화하여 소개한다.


# 문제 상황 #
리액트에서 기본적으로 form을 핸들링하는 패턴은 아래와 같다.

```jsx
import React from 'react'

class SignUpForm extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      email: '',
      password: '',
      passwordConfirm: ''
    }
    this._handleOnChangeInput = this._handleOnChangeInput.bind(this)
  }
  _handleOnChangeInput (e) {
    const { name, value } = e.target
    this.setState({ [name]: value })
  }
  render () {
    const { email, password, passwordConfirm } = this.state
    <form>
      <input
        type='text'
        name='email'
        placeholder='이메일주소'
        value={email}
        onChange={this._handleOnChangeInput}
      />
      <input
        type='password'
        name='password'
        placeholder='비밀번호'
        value={password}
        onChange={this._handleOnChangeInput}
      />
      <input
        type='password'
        name='passwordConfirm'
        placeholder='비밀번호 확인'
        value={passwordConfirm}
        onChange={this._handleOnChangeInput}
      />
      <button>Sign Up</button>
    </form>
  }
}
```
form handling과 관련되지 않은 부분을 생략한다면 View의 코드는 위와 같은 형태일 것이다. 이렇게 한 페이지 작성하는 것은 별 어려움이 없으나, form 요소가 들어가는 페이지가 많을 경우에는 굉장히 귀찮은 작업이다. 때때로 노가다 작업으로 느껴진다. form 요소가 들어가게 되면 항상 공통적으로 작성되는 부분이 존재하기 때문이다. 바로 `constructor()` 함수의 `this.state...` 부분과, `_handleOnChageInput()` 함수이다. 작업을 하다보니 아래와 같은 점에서 문제가 되었다.


1. `this.state`에서 form과 관련된 객체를 따로 뺄 경우, `_handleOnChangeInput()` 함수 구현이 많이 귀찮아졌다. 예를들어 `this.state.form`이라는 객체를 form의 필드들과 바인딩 시킬경우, `this.setState({ form: Object.assign({}, this.state.form, { [name]: value })})`와 같은 식이다. 복잡할 경우에는 [immutable.js][immutable]와 같은 라이브러리를 사용해야 할 경우도 있다.
2. validation이 필요할 경우에 문제는 더욱 커진다. 노가다 작업이 괴롭다.
3. 필드 수가 많은 경우 전체 코드에서 form handling에 관련된 코드가 차지하는 비중도 커진다.
4. Pure Component로 작성할 수 있는 간단한 컴포넌트도 state로 인해 생성자와 state를 작성해야 한다.
5. 무엇보다 같은 패턴의 함수들이 여러 컴포넌트에 걸쳐 작성되어야 한다는게 심리적으로 납득하기 힘들다.


위와 같은 이유로 인해 form handling과 관련된 관심사를 분리할 수 있는 HOC를 만들어보기로 했다.

# form handling HOC 구현 #

만들게 될 HOC의 스펙은 다음과 같다.


1. view에 랜더링되는 form의 각 필드들과 바인딩되는 state를 가지고 있고, 이를 view에 `formData`라는 prop으로 전달
2. view의 input의 `onChange` 이벤트에 따라 HOC의 state를 변경해 줄 `_handleOnChangeInput` 함수를 가지고 있고, 이를 view에 `onChangeInput`이라는 prop으로 전달
3. HOC 생성 시 `formConfig` 객체를 인자로 전달받아 state를 구성 (필드명, 초기값)


원리만 이해할 수 있도록 아주 간단한 스펙으로 설정해보았다. 실제로는 validation 기능까지 붙여서 프로젝트에 사용하고 있다. 아래 코드가 HOC 코드이다.

**withFormData.js**
```jsx
import React from 'react'
import PropTypes from 'prop-types'

export default formConfig => ComposedComponent => {
  class withFormData extends React.Component {
    constructor (props) {
      super(props)
      const initialState = {}
      formConfig.forEach(config => {
        initialState[config.inputName] = config.defaultValue || ''
      })
      this.state = initialState
      this._handleOnChangeInput = this._handleOnChangeInput.bind(this)
    }
    _handleOnChangeInput (e) {
      const { value, name } = e.target
      this.setState({ [name]: value })
    }
    render () {
      return <ComposedComponent onChangeInput={this._handleOnChangeInput} formData={this.state} {...this.props} />
    }
  }
  return withFormData
}
```

아래는 `withFormData.js`를 사용하여 `SignInView.js`를 감싸는 `SignInContainer.js`이다.

**SignInContainer.js**
```jsx
import withFormData from 'hocs/withFormData'
import SignInView from '../components/SignInView'

const formConfig = [
  {
    inputName: 'email',
    defaultValue: ''
  },
  {
    inputName: 'password',
    defaultValue: ''
  },
  {
    inputName: 'passwordConfirm',
    defaultValue: ''
  },
]

const wrappedSignInView = withFormData(formConfig)(SignInView)

export default wrappedSignInView
```

이로 인해 `onChangeInput`함수와 `formData`라는 state가 `SignInView`에 `props`로 주입되었다. 이제 이 prop들을 사용하는 View를 만들면 된다. 간단하게 Pure Component로 구현해보겠다.

**SignInView.js**
```jsx
import { PureComponent } from 'react'
import PropTypes from 'prop-types'

export default class SignInView extends PureComponent {
  static propTypes = {
    formData: PropTypes.object.isRequired,
    onChangeInput: PropTypes.func.isRequired
  }
  render () {
    const { formData, onChangeInput } = this.props
    return (
      <form>
        <input
          type='text'
          name='email'
          placeholder='이메일주소'
          value={formData.email}
          onChange={onChangeInput}
        />
        <input
          type='password'
          name='password'
          placeholder='비밀번호'
          value={formData.password}
          onChange={onChangeInput}
        />
        <input
          type='password'
          name='passwordConfirm'
          placeholder='비밀번호 확인'
          value={formData.passwordConfirm}
          onChange={onChangeInput}
        />
        <button>Sign Up</button>
      </form>
    )
  }
}
```

Component대신 PureComponent를 상속하여 `shouldComponentUpdate`호출 시 props를 `shallowCompare`해주는 컴포넌트를 구현하였다. 이처럼 stateless하게 컴포넌트를 구성하면 코드가 간결해진다. 또한 성능이나 side effect 측면에서도 유리하다. 실제 프로젝트에서는 `defaultValue`를 redux store에서 받은 전역 props에서 받아서 설정할 수 있는 부분과, validation 설정 부분이 추가되어서 더욱 복잡해졌지만, 기본적인 골자는 이와 같다. 한 번만 잘 만들어 놓으면 어떤 컴포넌트에서라도 너무나 편리하게 사용할 수 있으니, 나만의 form handling HOC를 꼭 만들어보자.

[지난포스팅]: https://eunvanz.github.io/react/2017/11/05/React-Higher-Order-Component-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0/
[immutable]: https://facebook.github.io/immutable-js/
