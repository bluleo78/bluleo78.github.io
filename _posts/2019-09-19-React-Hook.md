---
title: React Hook
categories: 
  - React
---

React Hook 를 사용하면 함수 컴포넌트에서 로컬 상태, 컨텍스트, 사이드이펙트 처리 등의 다양한 기능을 사용할 수 있는 방법을 제공한다. 그리고 이 경우 클래스를 이용해서 구현하는 것보다 더 간결하게 구현할 수 있다고 한다.

React 의 다음 몇가지 문제를 해결할 수 있다고 한다.
* 상태 관련 로직을 재사용하기 어려움
* 복잡한 컴포넌트는 이해하기 힘듦
* JavaScript 클래스는 여전히 어려운 개념임 => 함수 컴포넌트를 사용할 수 있어야 함.

상태 관련 로직에 담은 함수를 컴포넌트로부터 분리하여 컴포넌트를 간결한 함수 컴포넌트로 변환할 수 있고
분리된 함수는 재사용할 수 있다!


예제부터 살펴보자
---

```
import React, { useState } from 'react';

function Example() {

  // 로컬 상태 하나 추가(기본값은 0))
  // 'count' 로 상태 값을 참조하고, 'setCount' 로 상태 값을 변경하고자 함
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      
      // 기존 'count' 상태 값에 1 더해 'setCount' 를 이용하여 상태 변경
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
* 'userState' 를 훅(Hook) 이라고 부르고 있음


State Hook = useState
---
함수 컴포넌트에서 로컬 상태를 사용할 수 있게 함.
처음 호출 시 디폴트 값으로 상태를 만들고, 그 상태 값과 상태 업데이트 함수를 배열로 리턴
다음 호출부터는 저장된 상태 값과 업데이트 함수를 리턴함.

```
function ExampleWithManyStates() {
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  ...
}
```

Effect Hook = useEffect
---
함수 컴포넌트에서 컴포넌트 렌더링 이후 추가적인 작업을 지시할 수 있음
렌더링 이후의 작업을 함수로 넘김

```
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 사이드 이펙트 사용
  useEffect(() => {
    // 브라우저 문서의 타이틀을 업데이트
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
* 클릭할 때마다 카운트 로컬 상태를 증가시키고,
* 렌더링된 이후
* 브라우져의 문서 타이틀에 카운트를 포함하는 문구로 설정하고 있다.

사이드 이펙트 함수에서 리턴 값으로 함수(클린업 함수)를 넘기면, 렌더링 이후 이펙트 함수 수행 전 클린업 작업을 지정할 수 있습니다.

아래 예제는 친구 상태 구독과 구독 해제가 같은 훅 함수에 정의되어 있어 더 이해가 쉽습니다.
```
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);

    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```
* 렌더링 후에 구독 API 가 호출되고 컴포넌트 언마운트/렌더링 전에 구독 해제 API 가 호출됨


사이드 이펙트 함수가 조건부로 수행할 수 있음.
앞선 예제에서 다시 렌더링 할 때마다 구독 해제, 다시 구독하는 비효율적 작업이 반복되는데, 사이드 이펙트 실행 조건를 걸면 이를 방지할 수 있음.
```
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
...
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);

    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  }, [props.friend.id]);
...
}
```
* 친구 ID(props.friend.id) 가 바뀌지 않으면 사이드 이펙트 코드를 실행하지 않는다.

Hook 사용 시 주의할 점
---
* 함수 컴포넌트의 최상위에 위치해야 함. 반복문, 조건문, 내부 함수 등에서 사용하면 안됨.
  * 왜냐하면 훅의 실행 순서가 그 훅이 지정하는 상태 혹은 이펙트와 매핑되기 때문
* 함수 컴포넌트 혹은 다른 커스텀 훅에서만 호출되어야 함.

Custom Hook
---
먼저 'use' 프리픽스를 사용하여 커스텀 훅을 생성.
```
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

이 커스텀 훅을 아래 처럼 서도 다른 컴포넌트에서 재사용할 수 있음 
```
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

```
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

다른 Hook
---

### useContext
복잡한 컨텍스트 컨슈머(Consumer)를 사용하지 않고 컨텍스트를 쉽게 사용할 수 있게 함.
```
function Example() {
  const locale = useContext(LocaleContext);
  const theme = useContext(ThemeContext);
  // ...
}
```

### useReducer
단순 로컬 상태 업데이트가 아닌 액션에 따른 복잡한 상태 업데이트가 필요한 경우 리듀서를 사용할 수 있음.
```
function todosReducer(state, action) {
  switch (action.type) {
    case 'TODO':
      return {todos: [...state.todos, action.message]}
    ...
  }
}

function Todos() {
  const [state], dispatch] = useReducer(todosReducer, []);
  ...
  return (
    <>
      <ul>
        {state.todos.map((todo) => ...)}
      </ul>
      <Button onClick={() => dispatch({type:'ADD', message })}>
    </>
  )
}
```

### useCallback
함수 컴포넌트는 함수를 실행할 때마다 함수를 생성하는데, 이 오버헤드를 줄이기 위해서 이 훅을 제공하는 훅임
뒤에 의존성 배열에 값이 변경되는 경우에만 새로운 콜백을 리턴함.
```
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

### useMemo
값을 메모이제이션해서 처리하는 훅도 제공하고 있음.
마찬가지로 뒤에 의존성 배열 값이 변경되는 경우만 함수를 실행하여 값을 다시 계산하고, 그렇지 않으면 이전에 계산된 값을 리턴
```
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

### useRef
React.createRef() 와 같은 역할도 제공하고 있음
```
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` points to the mounted text input element
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

위에서 만약에 'input' DOM 이 변경이 된 경우 이를 인지하기 위해서는 useCallback 을 대신 사용할 수 있음
```
function MeasureExample() {
  const [height, setHeight] = useState(0);

  const measuredRef = useCallback(node => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
}
```

### useImperativeHandle, forwardRef
현재 함수 컴포넌트를 'ref' 로 참조했을 때, 해당 레퍼런스에서 사용할 수 있는 메소드를 제공함

아래 예제 처럼 함수 컴포넌트의 2번째 아귀먼트에 ref 를 넘기고,
'useImperativeHandle'로 ref 에 대한 핸들러를 작성.
최종적으로 이 함수를 'forwardRef' 감싼 컴포넌트를 리턴.(HOC)
```
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

이 컴포넌트에 사용할 때, 컴포넌트의 속성에 ref 설정하고 사용하면 됨.
```
function myFancy(props) {
  const inputRef = useRef(null);
  const onButtonClick = () => {
    inputRef.current.focus();
  };
  return (
    <>
      <FancyInput ref={inputRef} />
      <button onClick={onButtonClick}>Focus the FancyInput</button>
    </>
  );
}
```

### useLayoutEffect
전체 DOM 변경후 수행된다고 함.




참고
---
* <https://ko.reactjs.org/docs/hooks-overview.html>