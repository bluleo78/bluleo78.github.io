---
title: React Redux Reselect
categories: 
  - React
---

'reselect' 라이브러리는 스토어에서 계산된 값을 가져올 때, 이미 같은 조건에서 계산된 값이 있으면 그 값을 리턴하는 단순한 메모이제이션 라이브러리이다.

Redux 에서는 스토어의 상태 변경 시 스토어의 상태를 가져와 이를 props 로 변환하고, props 변경 사항이 있는 경우 컴포넌트를 렌더링한다.
이 때 'reselect' 를 사용하여 props 변환을 하면 메모이제이션이 사용되기 때문에 더 빠르게 처리할 수 있다.


예제부터 살펴보자
---
아래 예제는 아이템의 가격의 총합을 가져오는 selector 함수를 생성한 것이다.

```
import { createSelector } from 'reselect'

const myItemTotalCostSelector = createSelector(
  state => state.items,
  (items) => items.reduce((acc, item) => acc + item.cost, 0)
)

const state = { items: [ {name: 'A', cost:1}, {name: 'B', cost:2}]};

// 첫번째 호출에서는 아이템의 총합을 계산하여 리턴한다.
int totalCost1 = myItemTotalCostSelector(state);

// 두번째 호출에서는 state.items 가 이전과 같으므로 이전에 계산된 값을 리턴한다.
int totalCost2 = myItemTotalCostSelector(state);
```

이런 방식이 Redux 에서 아래처럼 사용될 수 있다.
```
import MyComponent from '../components/MyComponent'
import { createSelector } from 'reselect'

const myItemTotalCostSelector = createSelector(
  state => state.items,
  (items) => items.reduce((acc, item) => acc + item.cost, 0)
)

const mapStateToProps = (state) => {
  return {
    items: state.items,
    totalCost: myItemTotalCostSelector(state),
  }
}

export default MyComponent = connect(mapStateToProps)(MyComponent)
```

기본 Selector 함수 생성
---
아래는 조회 조건(state.visibilityFilter) 조건에 맞게 할일 목록을 구하는 Selector 함수이다.
```
import { createSelector } from 'reselect'

const getVisibilityFilter = (state) => state.visibilityFilter
const getTodos = (state) => state.todos

export const getVisibleTodos = createSelector(
  [ getVisibilityFilter, getTodos ],
  (visibilityFilter, todos) => {
    switch (visibilityFilter) {
      case 'SHOW_ALL':
        return todos
      case 'SHOW_COMPLETED':
        return todos.filter(t => t.completed)
      case 'SHOW_ACTIVE':
        return todos.filter(t => !t.completed)
    }
  }
)
```

여기서 [] 안에 기술되는 함수 아귀먼트를 inputSelector 함수라고 하고 마지막 아귀먼트를 결과 함수라고 한다.
inputSelector 의 결과 값이 이전 결과 값과 같다면 결과 함수를 호출하지 않고 캐시된 값을 리턴한다.


복합 Selector 함수 생성
---
아래 예제에서는 위에서 생성한 Selector 함수(getVisibleTodos)를 이용하여 다른 Selector 함수를 생성하고 있다.
```
const getKeyword = (state) => state.keyword

const getVisibleTodosFilteredByKeyword = createSelector(
  [ getVisibleTodos, getKeyword ],
  (visibleTodos, keyword) => visibleTodos.filter(
    todo => todo.text.includes(keyword)
  )
)
```

멀티 아귀먼트 Selector 생성
---
Redux 에서 스토어 상태(State) 외에 props 까지 받는 Selector 함수를 생성하려면 다음과 같이 하면 된다.
```
import { createSelector } from 'reselect'

const getVisibilityFilter = (state, props) =>
  state.todoLists[props.listId].visibilityFilter

const getTodos = (state, props) =>
  state.todoLists[props.listId].todos

const getVisibleTodos = createSelector(
  [ getVisibilityFilter, getTodos ],
  (visibilityFilter, todos) => {
    switch (visibilityFilter) {
      case 'SHOW_COMPLETED':
        return todos.filter(todo => todo.completed)
      case 'SHOW_ACTIVE':
        return todos.filter(todo => !todo.completed)
      default:
        return todos
    }
  }
)

const mapStateToProps = (state, ownProps) => {
  return {
    visibleTodos: getVisibleTodos(state, ownProps),
  }
}

export default VisibleTodoList = connect(makeMapStateToProps)(TodoList)

export default VisibleTodoList;
```

Selector 재사용 문제
---
여러 컴포넌트에서 같은 Selector 함수를 사용하면 캐시가 문제가 발생한다.
예를 들어 아래처림 위 VisibleTodoList 컴포넌트가 여러 개 사용되는 경우는 메모이제이션이 제대로 동작되지 않는다.
```
const App = () => (
  <div>
    <VisibleTodoList listId="1" />
    <VisibleTodoList listId="2" />
    <VisibleTodoList listId="3" />
  </div>
)
```
* 'props.listId' 가 순차적으로 1, 2, 3 변하기 때문에 'getVisibilityFilter', 'getTodos' 는 항상 이전과 다른 값을 리턴하게 된다.
* 따라서 'getVisibleTodos' 는 항상 재계산이 일어난다.

이런 경우 Selector 재사용하는 것이 아니라 컴포넌트마다 새로 생성해줘야 한다.

```
import { createSelector } from 'reselect'

const getVisibilityFilter = (state, props) =>
  state.todoLists[props.listId].visibilityFilter

const getTodos = (state, props) =>
  state.todoLists[props.listId].todos

const makeGetVisibleTodos = () => {
  return createSelector(
    [ getVisibilityFilter, getTodos ],
    (visibilityFilter, todos) => {
      switch (visibilityFilter) {
        case 'SHOW_COMPLETED':
          return todos.filter(todo => todo.completed)
        case 'SHOW_ACTIVE':
          return todos.filter(todo => !todo.completed)
        default:
          return todos
      }
    }
  )
}

const makeMapStateToProps = () => {
  const getVisibleTodos = makeGetVisibleTodos()
  const mapStateToProps = (state, props) => {
    return {
      todos: getVisibleTodos(state, props)
    }
  }
  return mapStateToProps
}
const VisibleTodoList = connect(makeMapStateToProps)(TodoList)

export default VisibleTodoList
```

참고
---
* <https://github.com/reduxjs/reselect>