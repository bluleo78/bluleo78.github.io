---
layout: post
title: React 프로젝트 기본 모듈 구성
---


immer
---
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