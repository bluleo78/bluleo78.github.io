---
layout: post
title: React 기본
---


함수형 컴포넌트
---
리액트 컴포넌트의 로컬 스태이트의 관리를 쉽게 하는 필수 유틸

```
function Square(props) {
  return (
    <button className="square" onClick={props.onClick}>
      {props.value}
    </button>
  );
}
```

참고
---
* </>
