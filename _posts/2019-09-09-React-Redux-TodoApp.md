---
layout: post
title: React Redux TodoApp 따라하기
---

Redux 를 사용하여 할일을 관히하는 TodoApp 을 만드는 과정을 따라해본다.

컴포넌트 설계
---
* TodoApp - 최상위 컴포넌트
  * AddTodo - 할일 추가 - Redux 스토어 연결
  * VisibleTodoList - 할일 목록 컨테이너 - Redux 스토어 연결
    * TodoList - 할일 목록
      * Todo - 할일
  * Footer - 하위 풋터
    * FilterLink - 필터 링크 컨테이너 - Redux 스토어 연결
      * Link - 링크


index.js
---
스토어를 생성하고 'Provider' 컴포넌트를 이용하여 전체 컴포넌트를 감싼다

<details>
  <summary>Click to expand!</summary>

```
import React from 'react';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import App from './containers/App';
import todoApp from './reducers';

let store = createStore(todoApp);

let rootElement = document.getElementById('root');
React.render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
);
```
</details>


actions.js
---
액션과 액션 생성자를 정의한다

<details>
  <summary>Click to expand!</summary>

```
export const ADD_TODO = 'ADD_TODO';
export const COMPLETE_TODO = 'COMPLETE_TODO';
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER';

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
};

export function addTodo(text) {
  return { type: ADD_TODO, text };
}

export function completeTodo(index) {
  return { type: COMPLETE_TODO, index };
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter };
}
```
</details>



reducers.js
---
리듀서 정의한다.
각각의 하위 상태에 대한 리듀서를 만들고, combineReducers 로 조합해서 리턴

<details>
  <summary>Click to expand!</summary>

```
import { combineReducers } from 'redux';
import { ADD_TODO, COMPLETE_TODO, SET_VISIBILITY_FILTER, VisibilityFilters } from './actions';
const { SHOW_ALL } = VisibilityFilters;

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return action.filter;
  default:
    return state;
  }
}

function todos(state = [], action) {
  switch (action.type) {
  case ADD_TODO:
    return [...state, {
      text: action.text,
      completed: false
    }];
  case COMPLETE_TODO:
    return [
      ...state.slice(0, action.index),
      Object.assign({}, state[action.index], {
        completed: true
      }),
      ...state.slice(action.index + 1)
    ];
  default:
    return state;
  }
}

const todoApp = combineReducers({
  visibilityFilter,
  todos
});

export default todoApp;
```
* 두 가지 상태를 하위 상태를 정의하고 있다. visibilityFilter, todos
* 'visibilityFilter' 상태
  * 'SET_VISIBITITY_FILTER' 액션이면 해당 액션의 filter 속성을 리턴
  * 그렇지 않으면 원래 상태 리턴
* 'todo' 상태
  * 배열 타입
  * 'ADD_TODO' 액션이면, 배열 뒤에 해당 액션으로부터 할일을 생성해서 추가
  * 'COMPLETE_TODO' 액션이면, 해당 액션의 인덱스를 찾아서 completed 값을 true 로 변경
</details>


App.js
---
Redux 상태로부터 props 를 가져와서 하위 컴포넌트를 렌더링하고,
하위 컴포넌트의 이벤트 콜백에서 액션을 생성하여 Redux 에 디스패치하는 형식이다.

<details>
  <summary>Click to expand!</summary>

```
import React from 'react';
import PropTypes from 'prop-types';
import { connect } from 'react-redux';
import { addTodo, completeTodo, setVisibilityFilter, VisibilityFilters } from '../actions';
import AddTodo from '../components/AddTodo';
import TodoList from '../components/TodoList';
import Footer from '../components/Footer';

class App extends React.Component {
  render() {
    const { dispatch, visibleTodos, visibilityFilter } = this.props;
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            dispatch(addTodo(text))
          } />
        <TodoList
          todos={visibleTodos}
          onTodoClick={index =>
            dispatch(completeTodo(index))
          } />
        <Footer
          filter={visibilityFilter}
          onFilterChange={nextFilter =>
            dispatch(setVisibilityFilter(nextFilter))
          } />
      </div>
    );
  }
}

App.propTypes = {
  visibleTodos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  })),
  visibilityFilter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
};

function selectTodos(todos, filter) {
  switch (filter) {
  case VisibilityFilters.SHOW_ALL:
    return todos;
  case VisibilityFilters.SHOW_COMPLETED:
    return todos.filter(todo => todo.completed);
  case VisibilityFilters.SHOW_ACTIVE:
    return todos.filter(todo => !todo.completed);
  default:
    return todos;
  }
}

function select(state) {
  return {
    visibleTodos: selectTodos(state.todos, state.visibilityFilter),
    visibilityFilter: state.visibilityFilter
  };
}

export default connect(select)(App);
```
* 'select' 함수에서 Redux 상태로부터 props 를 매핑을 만들고
* ''connect' 에서 이를 'App' 컴포넌트와 연결
* 이 과정을 통해서 'App' 컴포넌트는 'dispatch', 'visibleTodos', 'visibilityFilter' 를 props 전달받음
* 위 props 를 이용하여 하위 컴포넌트를 렌더링
</details>



Todo.js
---
<details>
  <summary>Click to expand!</summary>

```
import React from 'react';
import PropTypes from 'prop-types';

export default class Todo extends React.Component {
  render() {
    return (
      <li
        onClick={this.props.onClick}
        style={{
          textDecoration: this.props.completed ? 'line-through' : 'none',
          cursor: this.props.completed ? 'default' : 'pointer'
        }}>
        {this.props.text}
      </li>
    );
  }
}

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  text: PropTypes.string.isRequired,
  completed: PropTypes.bool.isRequired
};
```
* 'li' 태그를 이용하여 리스트 목록을 구성하고 있고,
* 'onClick' 이벤트 핸들러를 지원
* 'completed', 'text' 받아서 할일을 꾸미고 있다
</details>


TodoList.js
---
<details>
  <summary>Click to expand!</summary>

```
import React from 'react';
import PropTypes from 'prop-types';

import Todo from './Todo';

export default class TodoList extends React.Component {
  render() {
    return (
      <ul>
        {this.props.todos.map((todo, index) =>
          <Todo {...todo}
                key={index}
                onClick={() => this.props.onTodoClick(index)} />
        )}
      </ul>
    );
  }
}

TodoList.propTypes = {
  onTodoClick: PropTypes.func.isRequired,
  todos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  }).isRequired).isRequired
};
```
* 'ul' 태그를 이용하여 목록 구성
* 'todos' props 를 받아 하위 Todo 컴포넌트을 렌더링
* 'onTodoClick' props 를 하위 Todo 컴포넌트의 onClick 이벤트 콜백을 받아 호출
</details>


예제 코드에 대해서
---
* 'App' 컴포넌트를 상태 관리만 담당하는 Container 컴포넌트와 UI 를 구성을 담당하는 Presentational 컴포넌트로 분리하면 좋을 듯하다



참고
---
* <https://deminoth.github.io/redux>