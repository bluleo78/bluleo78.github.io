---
title: React Redux RedditApiApp 따라하기
categories: 
  - React
---

Redux 를 사용하여 비동기 액션을 사용하는 Reddit API 사용하여 글을 가져오는 앱을 만드는 과정을 따라해본다.

컴포넌트 설계
---
* AsyncApp - Redux 스토어 연결
  * AsyncApp - Reddit 조회 화면
    * Picker - 주제 선택
    * Posts - 선택된 주제 관련 글 목록


index.js
---
DOM 에 'Root' 컴포넌트 추가

```
import 'babel-polyfill';
import React from 'react';
import ReactDOM from 'react-dom';
import Root from './containers/Root';

ReactDOM.render(<Root />, document.getElementById('root'));
```


Root.js
---
스토어에 필요한 미들웨어를 설정하고, 앱(AysncApp)을 Provider 로 감싸서 렌더링한다.

```
import React from 'react';
import { Provider } from 'react-redux';
import configureStore from '../configureStore';
import AsyncApp from './AsyncApp';

const store = configureStore();

export default class Root extends React.Component {
  render() {
    return (
      <Provider store={store}>
        <AsyncApp />
      </Provider>
    );
  }
}
```


actions.js
---
액션과 액션 생성자를 정의한다.

```
import fetch from 'isomorphic-fetch';

export const REQUEST_POSTS = 'REQUEST_POSTS';
export const RECEIVE_POSTS = 'RECEIVE_POSTS';
export const SELECT_REDDIT = 'SELECT_REDDIT';
export const INVALIDATE_REDDIT = 'INVALIDATE_REDDIT';

export function selectReddit(reddit) {
  return {
    type: SELECT_REDDIT,
    reddit
  };
}

export function invalidateReddit(reddit) {
  return {
    type: INVALIDATE_REDDIT,
    reddit
  };
}

function requestPosts(reddit) {
  return {
    type: REQUEST_POSTS,
    reddit
  };
}

function receivePosts(reddit, json) {
  return {
    type: RECEIVE_POSTS,
    reddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  };
}

function fetchPosts(reddit) {
  return dispatch => {
    dispatch(requestPosts(reddit));
    return fetch(`http://www.reddit.com/r/${reddit}.json`)
      .then(req => req.json())
      .then(json => dispatch(receivePosts(reddit, json)));
  }
}

function shouldFetchPosts(state, reddit) {
  const posts = state.postsByReddit[reddit];
  if (!posts) {
    return true;
  } else if (posts.isFetching) {
    return false;
  } else {
    return posts.didInvalidate;
  }
}

export function fetchPostsIfNeeded(reddit) {
  return (dispatch, getState) => {
    if (shouldFetchPosts(getState(), reddit)) {
      return dispatch(fetchPosts(reddit));
    }
  };
}
```
* selectReddit: 레딧 주제 선택 액션 생성
* invalidateReddit: 해당 레딧 주제에 포스트 무효화
* requestPosts: 해당 레딧 주제에 대한 글 조회요청
* receivePosts: 해당 레딧 주제에 대한 글 업데이트 요청
* fetchPostsIfNeeded: 필요한 해당 레딧 주제에 글 조회 요청 액션(requestPosts)을 생성한 다음 사이트에서 데이터를 가져와 글 업데이트 액션(receivePosts)을 생성한다.


reducers.js
---
리듀서 정의한다. 각각의 하위 상태에 대한 리듀서를 만들고, combineReducers 로 조합해서 리턴한다.

```
import { combineReducers } from 'redux';
import {
  SELECT_REDDIT, INVALIDATE_REDDIT,
  REQUEST_POSTS, RECEIVE_POSTS
} from './actions';

function selectedReddit(state = 'reactjs', action) {
  switch (action.type) {
  case SELECT_REDDIT:
    return action.reddit;
  default:
    return state;
  }
}

function posts(state = {
  isFetching: false,
  didInvalidate: false,
  items: []
}, action) {
  switch (action.type) {
  case INVALIDATE_REDDIT:
    return Object.assign({}, state, {
      didInvalidate: true
    });
  case REQUEST_POSTS:
    return Object.assign({}, state, {
      isFetching: true,
      didInvalidate: false
    });
  case RECEIVE_POSTS:
    return Object.assign({}, state, {
      isFetching: false,
      didInvalidate: false,
      items: action.posts,
      lastUpdated: action.receivedAt
    });
  default:
    return state;
  }
}

function postsByReddit(state = { }, action) {
  switch (action.type) {
  case INVALIDATE_REDDIT:
  case RECEIVE_POSTS:
  case REQUEST_POSTS:
    return Object.assign({}, state, {
      [action.reddit]: posts(state[action.reddit], action)
    });
  default:
    return state;
  }
}

const rootReducer = combineReducers({
  postsByReddit,
  selectedReddit
});
export default rootReducer;
```
* 두 가지 상태를 하위 상태를 정의하고 있다.
* 'postsByReddit' 상태
  * 'INVALIDATE_REDDIT' 액션: 'didInvalidate' 을 참으로 만들어 필요하면 사이트에서 데이터를 가져올 수 있게
  * 'REQUEST_POSTS' 액션: 'isFetching' 을 참으로 만들어 데이터를 가져오고 있는 상태로 표시할 수 있게
  * 'RECEIVE_POSTS' 액션: 'isFetching' 을 거짓으로 만들어 가져온 데이터를 화면에 그릴 수 있게
* 'selectedReddit' 상태
  * 'SELECT_REDDIT' 액션: 선택한 주제로 변경할 수 있게


AsyncApp.js
---
Redux 상태로부터 props 를 가져와서 하위 컴포넌트를 렌더링하고,
하위 컴포넌트의 이벤트 콜백에서 액션을 생성하여 Redux 에 디스패치하는 형식이다.

```
import { connect } from 'react-redux';
import { selectReddit, fetchPostsIfNeeded, invalidateReddit } from '../actions';
import AsyncApp from '../components/AsyncApp';


function mapStateToProps(state) {
  const { selectedReddit, postsByReddit } = state;
  const {
    isFetching,
    lastUpdated,
    items: posts
  } = postsByReddit[selectedReddit] || {
    isFetching: true,
    items: []
  };

  return {
    selectedReddit,
    posts,
    isFetching,
    lastUpdated
  };
}


function mapDispatchToProps(dispatch) {
  return {
    onRefreshPosts: (selectedReddit) => {
      dispatch(fetchPostsIfNeeded(selectedReddit));
    },
    onChangeSelectedReddit:(selectedReddit) => {
      dispatch(selectReddit(selectedReddit));
    },
    onClickRefresh: (selectedReddit) => {
      dispatch(invalidateReddit(selectedReddit));
      dispatch(fetchPostsIfNeeded(selectedReddit));
    }
  };
}


export default connect(mapStateToProps, mapDispatchToProps)(AsyncApp);
```
* 'mapStateToProps' 함수에서 Redux 상태로부터 props 를 매핑을 만들고
  * selectedReddit: Redux 상태 그대로 매핑
  * posts : Redux 스토어의 'postsByReddit'의 해당 주제의 items 를 매핑
  * isFetching : Redux 스토어의 'postsByReddit'의 해당 주제의 isFetching 을 그대로 매핑
  * lastUpdate : Redux 스토어의 'postsByReddit'의 해당 주제의 lastUpdated 을 그대로 매핑
* 'mapDispatchToProps' 함수에서 이벤트 콜백 props 함수에 대한 액션 디스패치 로직을 매핑
  * onRefreshPosts: 레딧 글 목록을 가져옴
  * onChangeSelectedReddit: 레딧 주제 변경 알림
  * onClickRefresh: Refresh 를 클릭했을 때, 레딧 포스트를 무효화하고 현재 주제에 대한 글 목록을 가져옴
* 이를 'connect' 에서 이를 'AsyncApp' Presentational 컴포넌트와 연결


AsyncApp.js
---
앱 Presetational 컴포넌트

```
import React from 'react';
import PropTypes from 'prop-types';
import Picker from './Picker';
import Posts from './Posts';

class AsyncApp extends React.Component {

  componentDidMount() {
    this.props.onRefreshPosts(this.props.selectedReddit);
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.selectedReddit !== this.props.selectedReddit) {
      this.props.onRefreshPosts(nextProps.selectedReddit);
    }
  }

  handleClickRefresh(e) {
    e.preventDefault();

    this.props.onClickRefresh(this.props.selectedReddit);
  }

  render () {
    const { selectedReddit, posts, isFetching, lastUpdated } = this.props;
    return (
      <div>
        <Picker value={selectedReddit}
                onChange={(nextReddit) => this.props.onChangeSelectedReddit(nextReddit)}
                options={['reactjs', 'frontend']} />
        <p>
          {lastUpdated &&
            <span>
              Last updated at {new Date(lastUpdated).toLocaleTimeString()}.
              {' '}
            </span>
          }
          {!isFetching &&
            <a href='#'
               onClick={(e) => this.handleClickRefresh(e)}>
              Refresh
            </a>
          }
        </p>
        {isFetching && posts.length === 0 &&
          <h2>Loading...</h2>
        }
        {!isFetching && posts.length === 0 &&
          <h2>Empty.</h2>
        }
        {posts.length > 0 &&
          <div style={{ opacity: isFetching ? 0.5 : 1 }}>
            <Posts posts={posts} />
          </div>
        }
      </div>
    );
  }
}

AsyncApp.propTypes = {
  selectedReddit: PropTypes.string.isRequired,
  posts: PropTypes.array.isRequired,
  isFetching: PropTypes.bool.isRequired,
  lastUpdated: PropTypes.number,
  onRefreshPosts: PropTypes.func.isRequired,
  onChangeSelectedReddit: PropTypes.func.isRequired,
  onClickRefresh: PropTypes.func.isRequired
};

export default AsyncApp;
```
* 컴포넌트 마운트 시에 현재 레딧 주제에 대한 글 목록을 패치&업데이트
* 컴포넌트의 'selectedRedit' 가 변할 때, 해당 레딧 주제에 대한 글 목록을 패치&업데이트
* 상단에는 주제 선택을 위한 Picker 컴포넌트
* 하단에는
  * 업데이트 날짜,
  * Refresh 링크
  * 데이터 패치 중 'Loading...' 문구
  * 데이터 가 없을 때 'Empty.' 문구
  * 글 목록, 패치 중엔 경우는 반투명하게 (opacity: 0.5)
* props 타입 체크


Picker.js
---
레딧 주제 선택을 위한 콤보 박스 제공

```
import React from 'react';
import PropTypes from 'prop-types';

export default class Picker extends React.Component {
  render () {
    const { value, onChange, options } = this.props;

    return (
      <span>
        <h1>{value}</h1>
        <select onChange={e => onChange(e.target.value)}
                value={value}>
          {options.map(option =>
            <option value={option} key={option}>
              {option}
            </option>)
          }
        </select>
      </span>
    );
  }
}

Picker.propTypes = {
  options: PropTypes.arrayOf(
    PropTypes.string.isRequired
  ).isRequired,
  value: PropTypes.string.isRequired,
  onChange: PropTypes.func.isRequired
};
```
* 'select', 'option' 태그 사용


Posts.js
---
레딧 글 목록 표시

```
import React from 'react';
import PropTypes from 'prop-types';

export default class Posts extends React.Component {
  render () {
    return (
      <ul>
        {this.props.posts.map((post, i) =>
          <li key={i}>{post.title}</li>
        )}
      </ul>
    );
  }
}

Posts.propTypes = {
  posts: PropTypes.array.isRequired
};
```
* 'ul' 태그를 이용하여 목록 구성


참고
---
* <https://deminoth.github.io/redux>