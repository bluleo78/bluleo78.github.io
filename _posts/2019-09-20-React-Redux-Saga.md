---
title: React Redux Saga
categories: 
  - React
---

Redux Saga 는 React 의 사이드 이펙트를 좀 더 쉽게 관리하고, 핸들링할 수 있는 미들웨어임.
Redux Thunk 와 마찬가지로 비동기 액션을 처리할수 있음.
Redux Thunk 와 다르게 사이드 이펙트를 담당하는 별도의 쓰레드처럼 동작함. 사실은 제너레이터임.
제너레이터 함수를 이터레이션하는 방법으로 쉽게 테스트 코드를 작성할 수 있다는 장점

예제부터 살펴보자
---
### actions.js
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
```

### sagas.js
```
import { call, put, takeEvery } from 'redux-saga/effects'
import { REQUEST_DATA, receiveDataFromServer } from './actions'

function* fetchDataFromServer(action) {
   try {
      const data = yield call(fetch, 'http://my.server/data');
      yield put(receiveDataFromServer(data));
   } catch (e) {
      // Error Handling
   }
}

function* mySaga() {
  yield takeEvery(REQUEST_DATA, fetchDataFromServer);
}

export default mySaga;
```

### App.js
```
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'
import reducer from './reducers'
import mySaga from './sagas'

// 사가 미들웨어를 생성
const sagaMiddleware = createSagaMiddleware()

// 스토어 생성 시 미들웨어 추가
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)

// 사가를 실행
sagaMiddleware.run(mySaga)
```

태스크 생성
---

### takeEvery
지정된 액션에 대해서 항상 태스크를 실행
```
yield takeEvery('액션이름', 태스크);
```

### takeLatest
항상 마지막 액션에 대해서만 태스크를 실행
만약에 실행중인 태스크가 있으면, 취소해버림
```
yield takeLatest('액션이름', 태스크);
```

함수 호출
---
### call
직접 함수를 호출하는 것이 아니라 어떤 함수를 호출하는지 상세를 리턴한다.
실제 호출은 Redux Saga 가 알아서 호출함
```
yield call(함수, 아귀먼트...)
yield call([객체, '메소드이름'], 아귀먼트...)
```

### apply
'call' 과 거의 유사한데, 아귀먼트를 배열로 넘긴다.
```
yield apply(함수, [아귀먼트...])
yield call([객체, '메소드이름'], [아귀먼트...])
```

액션 디스패치
---
### put
액션을 디스패치하는 상세를 리턴한다.
실제 디스패치는 Redux Saga 가 함.
```
yield put(액션)
```

Redux Saga vs Redux Thunk
---
* 둘다 비동기 액션을 지원하는 것은 마찬가지
* React Thunk 는 액션을 비동기적으로 수행하는 함수 자체를 디스패치하는 방식이고
* React Saga 는 특정 액션이 디스패치되었을 때 다른 액션을 비동기적으로 수행하는 함수를 실행하는 방식
* Promise vs Generator
* 테스트하기 쉽고, 취소도 가능하고


참고
---
* <https://react-redux.js.org/next/api/hooks>
* <https://velog.io/@dongwon2/Redux-Thunk-vs-Redux-Saga%EB%A5%BC-%EB%B9%84%EA%B5%90%ED%95%B4-%EB%B4%85%EC%8B%9C%EB%8B%A4->