---
layout: post
title: "[React] Higher Order Component 이해하기"
date: 2018-02-05 22:31:38 +0900
author: 방구석엔지니어
categories: react
tags: react hoc
cover: "/assets/react.jpeg"
---

# Higher Order Component 개요 #
Higher Order Component (이하 HOC)는 리액트에서 사용되는 일종의 디자인 패턴이다. HOC는 리액트 컴포넌트를 인자로 받아 리액트 컴포넌트를 반환한다. 기본적으로 아래와 같은 형태이다.
```jsx
export default ComposedComponent => {
  class HOC extends React.Component {
    const { ...props } = this.props
    render () {
      return <ComposedComponent {...props} />
    }
  }
  return HOC
}
```
위 코드를 템플릿으로 하여 갖가지 양념을 쳐서 특정한 기능을 수행하는 컴포넌트를 생산할 수 있다. 주로, 여러 컴포넌트에 반복적으로 들어가는 코드들을 HOC에 작성하면 된다. 즉, 여러 컴포넌트들이 가지고 있는 횡단관심사(Cross-cutting concerns)들을 구현하는 컴포넌트가 HOC이다.

스프링 AOP를 공부하면서 들었던 개념이 횡단관심사라는 개념인데, View에서의 횡단관심사는 감이 잘 잡히지 않는다. View에서의 횡단관심사들은 어떤것들이 있을까? 생각해보면 생각보다 많다. 그리고 실제로 작업을 하다보면 이러한 횡단관심사들이 View에서도 상당히 많다. 일단 생각나는 것을 떠올려보면,

- **유저 인증 로직:** 사용자가 로그인 상태인지, 권한이 있는 사용자인지에 따라 페이지를 보여주거나, 로그인 페이지로 이동시키거나, 권한이 없다는 안내 메시지를 띄우는 등, 사용자 상태에 따라 분기를 시키는 유저 인증 작업을 HOC에게 위임할 수 있다.
- **Dumb와 Smart 컴포넌트 분리:** Redux 패턴에서 권장하는 방식인 Dumb 컴포넌트(Presentational Component)와 Smart 컴포넌트(Container Component)를 분리하는데에 사용되고 있다. `react-redux`모듈에서는 `connect`라는 HOC를 제공하고 있으며, 이는 Dumb 컴포넌트와 Smart 컴포넌트를 분리할 수 있게 해준다. 이로 인해 컴포넌트의 재사용성과 유지보수 용이성을 높일 수 있다.
- **로딩 화면 표시:** 화면을 띄우기위한 데이터가 로딩 중일 때 로딩 화면을 표시하고, 데이터 로딩이 완료됐을 때 인자로 받은 컴포넌트를 보여주도록 하는 로직을 HOC가 담당하게 할 수 있다.

이러한 것들을 쉽게 떠올릴 수 있다. 좀 더 복잡하게 사용하면 폼 데이터와 state의 바인딩 등과 같이 더욱 다양한 상황에서 사용할 수 있다.

# Higher Order Component 구현 #
간단하게 HOC를 구현해 보는 예제이다. 사용자 로그인 상태일경우 페이지를 정상적으로 보여주고, 로그인이 돼있지 않을 경우 로그인 페이지로 이동하는 로직을 갖고 있는 HOC를 구현해보겠다.

그 전에 몇 가지 가정이 필요하다. 아래는 현재 내가 실제로 진행하고 있는 프로젝트의 환경과 같다.

- 사용자가 로그인 상태일 경우에는 redux store의 state에 user라는 키값으로 user 객체가 존재
- **needAuth.js:** 로그인 여부에 따라 페이지 이동 처리를 하는 HOC로, 로그인 상태가 아닐 시 `/sign-in` route로 이동
- **UserProfileContainer.js:** redux store와 뷰를 연결짓는 container 컴포넌트
- **UserProfileView.js:** 사용자 자신의 프로필을 조회하는 presentational 컴포넌트

**needAuth.js:**
```jsx
import React from 'react'
import PropTypes from 'prop-types'
import { connect } from 'react-redux'

export default ComposedComponent => {
  class needAuth extends React.Component {
    componentDidMount () {
      const { user } = this.props
      if (!user) this.context.router.push('/sign-in')
    }
    render () {
      const { user, ...props } = this.props
      if (!user) return <div>로그인 처리 중...</div>
      return <ComposedComponent user={user} {...props} />
    }
  }
  needAuth.contextTypes = {
    router: PropTypes.object.isRequired
  }
  needAuth.propTypes = {
    user: PropTypes.object
  }
  const mapStateToProps = state => {
    return {
      user: state.user
    }
  }
  return connect(mapStateToProps, null)(needAuth)
}
```

**UserProfileContainer.js:**
```jsx
import UserProfileView from '../components/UserProfileView'
import needAuth from 'hocs/needAuth'

export default needAuth(UserProfileView)
```

**UserProfileView.js:**
```jsx
import React from 'react'
import PropTypes from 'prop-types'

class UserProfileView extends React.Component {
  render () {
    const { user } = this.props
    return (
      <div>
        <p>name: {user.name}</p>
        <p>mobile: {user.mobile}</p>
        <p>hobby: {user.hobby}</p>
      <div>
    )
  }
}
UserProfileView.propTypes = {
  user: PropTypes.object.isRequired
}
export default UserProfileView
```

위와 같이 간단하게 HOC를 만들고 적용해보았다. 실제 프로젝트에 적용한 결과 아주 잘 작동한다.