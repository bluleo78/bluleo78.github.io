---
title: React Redux Thunk
categories: 
  - React
---

'Thunk' 라는 말은 나중에 수행될 혹은 다른 코드 앞/뒤에 주입되는 코드 조각이라는 의미라고 함.
Redux 의 Thunk 는 스토어에 액션 객체가 아닌 내부에서 액션을 디스패치하는 함수를 전달했을 때, 이를 실행시켜주는 미들웨어임.


예제부터 살펴보자
---
```
import fetch from 'isomorphic-fetch';
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers';


// 스토어 생성 시 Thunk 미들웨어를 추가
const store = createStore(rootReducer, applyMiddleware(thunk));
...
```

```
export const REQUEST_DATA = "REQUEST_DATA";
export const requestDataToServer = () => {
  return {
    type: REQUEST_DATA,
  };
};


export const RECEIVE_DATA = "RECEIVE_DATA";
export const receiveDataFromServer = (data)) => {
  return {
    type: RECEIVE_DATA,
    data,
  };
}

export const fetchDataFromServer = () => {
  return (dispatch, getState) => {
    dispatch(requestDataToServer()) 

    return fetch('http://my.server/data')
      .then(req => req.json())
      .then(data => dispatch(receiveDataFromServer(data)));
};
```
* 여기서 'fetchDataFromServer()' 를 디스패치하면 비동기적으로 'requestDataToServer()', 'receiveDataFromServer(data)' 액션이 디스패치가 됨.


참고
---
* <https://react-redux.js.org/next/api/hooks>