---
title: React 라우터 구성
---

기본 구성
---
```
// App.js

...
import { BrowserRouter as Router, Route, Link } from "react-router-dom";

function App() {
  return (
    <Router>
      <Route path="/" exact component={Home}>
      <Route path="/about" component={About}>
      <Route path={["/hi", "/hello"]} component={Hello}>
    </Router>
  );
}
...
```
* 기본적으로 'Router' 태그 아래 'Route' 태그를 사용하여 라우팅을 설정하는 방식이다
* URL 경로와 매치되는 경우 지정된 컴포넌트를 렌더링하는 방식이다
* 'exact' 는 해당 경로가 정확히 매치되야 라우팅된다.
* 경로는 배열로도 기술할 수 있다

중첩(Nested) 라우팅
---
특정 경로에 하위 경로가 존재하고 서로 공유되는 부분이 있는 경우 이를 처리할 수 있는 방법을 제공한다.
예를 들어 '/topics' 아래 '/topics/news' 와 '/topics/humors' 가 있는 경우 다음과 같이 기술할 수 있다.
```
function App() {
  return (
    <Router>
      <Route path="/topics" component={Topics}>
    </Router>
  );
}

function Topics(props) {
  return (
    <>
      <h1> Topics </h1>
      <Route path={`${props.match.url}/news`} component={News}>
      <Route path={`${props.match.url}/humors`} component={Humors}>
    </>
  );
}
```


경로 변경(Navigation)
---
경로 변경은 클릭했을 때 경로가 바뀌는 링크를 제공하거나 프로그래밍을 하여 경로를 바꾸는 방법이 있다

### 링크
```
import { Link } from 'react-router-dom'

<Link to="/about">About</Link>
```

### 경로 변경
<Redirect> 태그가 렌더링되는 경우 경로가 바뀐다.
```
import { Redirect } from 'react-router-dom'
...

render() {
  if (this.state.redirect) {
    return <Redirect to="/About"/>
  }

  return (
    <button onClick={() => this.setState({redirect:true})}>Redirect</button>
  );
}
```

RegExp 경로 기술
---
경로는 path-to-regexp 포맷 사용(<https://github.com/pillarjs/path-to-regexp>)
따라서 콜론(:)+이름을 사용하면 해당 구역에 대해 Regular Exp(([^\/]+?) 처리할 수 있음
이 경우 컴포넌트의 props.match.param 에 해당 이름에 대한 값을 구할 수 있음
```
<Route path="/jobs/:id" component={JobDetail}>
<Route path="/a/:id?" component={A}>
<Route path="/b/:id+" component={B}>
<Route path="/c/:id*" component={C}>
<Route path="/d/:id(\\d+)" component={D}>
```


참고
---
* <>