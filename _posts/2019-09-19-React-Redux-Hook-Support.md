---
title: React Redux Hook Support
categories: 
  - React
---

React Redux 에서 함수 컴포넌트에서 사용할 수 있는 훅(Hook)을 제공


예제부터 살펴보자
---

```
import React from 'react'
import { useSelector } from 'react-redux'

export const CounterComponent = () => {
  const counter = useSelector(state => state.counter)
  return <div>{counter}</div>
}
```
* Redux 스토어의 counter 값을 받아 화면에 출력하는 내용임

```
import React from 'react'
import { useSelector } from 'react-redux'

export const TodoListItem = props => {
  const todo = useSelector(state => state.todos[props.id])
  return <div>{todo.text}</div>
}
```
* Redux 스토어의 todos 목록 중 'id' props 에 해당하는 todo 를 받아 화면에 출력하는 내용임.


useSelector
---

Redux Selector 훅인 'useSelector'는 connect(mapStateToProps) 와 비슷한 역할을 한다.
* Redux 스토어에서 접근하여 값을 가져올 수 있고,
* Redux 스토어가 변경되었을 때, 해당 값이 변경된 경우 다시 컴포넌트를 렌더링을 하게 만듬.

다음은 useSelector 함수의 포맷
```
const result : any = useSelector(selector : Function, equalityFn? : Function)

```
* selector = state: any => value: any
  * 스토어 상태를 받아서 값을 리턴
* equalityFn = (value: any, value:any) => equality: boolean
  * selector 의 결과값과 이전 결과값을 비교하는 함수
  * 생략되면 '===' 비교

참고로 이전의 'connect' 의 'matStateToProps' 에서는 shallow 비교를 수행하니 이를 selector 로 변환하기 위해서는 아래처럼 'shallowEqual'을 사용하면 됨.
```
import { shallowEqual, useSelector } from 'react-redux'

const selectedData = useSelector(selectorReturningObject, shallowEqual)
```

함수 컴포넌트에서는 컴포넌트가 렌더링 될 때마다 'useSelector' 의 selector 함수가 생성이 된다. 이를 방지하기 위해서 컴포넌트 밖에 selector 함수를 정의해서 사용하는 것이 좋다.
그리고 selector 함수의 결과값을 메모이제이션하면 'reselect' 를 사용하면 값이 변경되지 않은 경우 기존 결과값을 리턴하기 때문에 '===' equalityFn 을 사용하여 불필요한 렌더링을 없앨 수 있다.
```
import React from 'react'
import { useSelector } from 'react-redux'
import { createSelector } from 'reselect'

const selectNumOfDoneTodos = createSelector(
  state => state.todos,
  todos => todos.filter(todo => todo.isDone).length
)

export const DoneTodosCounter = () => {
  const NumOfDoneTodos = useSelector(selectNumOfDoneTodos)
  return <div>{NumOfDoneTodos}</div>
}

export const App = () => {
  return (
    <>
      <span>Number of done todos:</span>
      <DoneTodosCounter />
    </>
  )
}
```
* 위에서는 state.todos 가 변경되지 않은 경우에 완료된 todo 아이템의 개수를 다시 계산하지 않고 이전 계산값을 리턴




참고
---
* <https://react-redux.js.org/next/api/hooks>