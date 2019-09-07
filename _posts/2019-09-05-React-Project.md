---
title: React 프로젝트 구성
---

프로젝트 생성
---
```
npx create-react-app 프로젝트이름
```


라우터(Router) 설정
---
라우터는 URL 경로에 맞는 컴포넌트를 로딩할 수 있게 하는 모듈이다

### 리액트 라운터 모듈 추가
```
npm install --save react-router-dom
```

### 라우트 파일 추가
```
// routes.js

import React from "react";
import { Route } from "react-router-dom";
// import Home from './pages/Home';

function routes() {
  return (
    <div>
      // <Route exact path="/" component={Home} />
    </div>
  );
}

export default routes;
```
* 라우트 파일에서는 경로에 해당하는 컴포넌트(주로 페이지 컴포넌트)를 설정해줌


### 라우트 이용하도록 App.js 수정
```
// App.js

import React from "react";
import { BrowserRouter as Router } from "react-router-dom";
import routes from './routes'

function App() {
  return (
    <Router>
      {routes}
    </Router>
  );
}

export default App; 
```


유용한 모듈
---
## immer
리액트 컴포넌트의 로컬 스태이트의 관리를 쉽게 하는 필수 유틸입니다.
특히 여러 개의 상태 값을 갖거나, 상태 값이 배열이나 다층 구조인 경우에 유용하다.

```
import produce from "immer"

class Board extends React.Component {

  constructor(props) {
    super(props);
    this.state = {
      names: ["Alex", "Mary"]
    }
  }

  handleClick() {
    const nextState = produce(this.state, state => {
      state.names[1] = "Paul";
    });

    this.setState(nextState);
  }
  ...
}
```


참고
---
* <>